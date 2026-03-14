---
name: hands-free
description: Use when the user invokes /hands-free to enable auto-accept mode for skill recommendations. Hands-off workflow that auto-proceeds with recommended options. Supports full/partial/off modes, learning sensitivity, and smart recommendations.
---

# Hands-Free

Auto-accept recommended options from any skill without pausing. Works with superpowers, custom skills, or any workflow with approval points.

## Commands

```
/hands-free              # full mode (recommended)
/hands-free full         # same — auto-accept all non-destructive points
/hands-free partial      # auto-accept design only, pause at execution
/hands-free off          # disable
/hands-free crazy-workspace [dir]  # approve everything scoped to [dir], except rm -rf * and rm -rf .git
/hands-free auto-commit on    # auto-commit changes at natural milestones
/hands-free auto-commit off   # disable auto-commit (default)
/hands-free learning <h/m/l>  # set learning sensitivity
/hands-free recommend    # show recommended settings based on usage
/hands-free log          # show session decisions
/hands-free status       # show current mode + learning level
```

## Recommended Setup

> `/hands-free full` + `/hands-free learning high`
>
> Best for experienced users who trust the defaults and want maximum speed. Auto-accepts everything non-destructive, learns your preferences after a single choice.

## Mode Behavior

| Approval Type | full | partial | off | crazy-workspace |
|---|---|---|---|---|
| Brainstorming approaches | auto | auto | ask | auto |
| Design approval | auto | auto | ask | auto |
| Execution method | auto | **ask** | ask | auto |
| Batch checkpoints | auto | **ask** | ask | auto |
| Phase transitions | auto | auto | ask | auto |
| Grep tool calls | auto | auto | ask | auto |
| Shell cmd scoped to current dir | auto | auto | ask | auto |
| `pyenv` commands | auto | auto | ask | auto |
| `git init` | auto | auto | ask | auto |
| `git add` | auto | auto | ask | auto |
| `cd` within workspace | auto | auto | ask | auto |
| Destructive actions | **ask** | **ask** | **ask** | auto (in target dir) |
| Git commit (auto-commit on) | auto | auto | ask | auto |
| Git push | **ask** | **ask** | **ask** | auto |
| `rm -rf *` | **ask** | **ask** | **ask** | **HARD STOP** |
| `rm -rf .git` | **ask** | **ask** | **ask** | **HARD STOP** |

Mode and learning can be combined: `/hands-free full` then `/hands-free learning high`. **Learning thresholds govern when preferences are recorded and applied; mode governs what gets auto-accepted when no preference exists.** They are independent axes.

## Core Rule

When active, MUST auto-proceed with the recommended option. Do NOT pause, present options, or wait. **Announce, don't ask:** "Going with approach 2 (recommended) — [reason]" and continue.

Applies to **any skill** — not just superpowers:
- Options with recommendation → pick it
- Approval to continue → approve
- Design/plan review → approve
- Checkpoint pause → continue

### When You Must Pause and Ask

In partial mode or at hard stops, when presenting options to the user via `AskUserQuestion`, mark the recommended option with a `markdown` preview panel — do NOT add "(Recommended)" to the label:

```
{
  "label": "Subagent-Driven",
  "description": "Dispatch fresh subagent per task with two-stage review",
  "markdown": "## Recommended\n\nBest for staying in this session with fast iteration.\n\n**Pros:** No context switch, review checkpoints automatic\n**Con:** More subagent invocations"
}
```

The `markdown` field is only visible when the option is focused — it surfaces the rationale without cluttering the label. Use it on the recommended option only.

### Superpowers-Specific Approval Points

| Skill | Approval Point | Auto Action |
|-------|---------------|-------------|
| brainstorming | 2-3 approach options | Pick recommended approach |
| brainstorming | Design section approval | Approve, continue to next |
| brainstorming | Final design approval | Approve, proceed to writing-plans |
| writing-plans | Execution method choice | Pick recommended method (full only) |
| executing-plans | Batch checkpoint | Continue to next batch (full only) |
| systematic-debugging | Phase transitions | Proceed through all phases |

## Shell Command Auto-Pass Rules

In `full`, `partial`, and `crazy-workspace` modes, auto-approve Bash/shell tool calls without asking when **any** of these conditions are met:

### Always auto-pass (regardless of paths)

- `pyenv` — any pyenv subcommand (`pyenv install`, `pyenv local`, `pyenv global`, etc.)
- `git init` — initializing a repo
- `git add` — staging files (not destructive)
- `cd` within the workspace — changing into any subdirectory of the current workspace

### Auto-pass when scoped to current directory

A shell command is **scoped to the current directory** if it contains no paths that escape the working directory. Auto-pass if the command does NOT contain:
- Absolute paths outside the current dir (e.g. `/etc`, `/usr`, `~/.ssh`, `/var`)
- Parent directory traversal (`../`) that exits the current dir
- System-wide write targets (`/usr/local/bin`, `/etc/hosts`, etc.)

If the command only references relative paths, current-dir files, or env vars scoped to the project, it is safe to auto-pass.

### Decision flow for shell commands

```dot
digraph {
    "Shell command" -> "Always-pass list?";
    "Always-pass list?" -> "Auto-approve" [label="yes (pyenv, git init, git add)"];
    "Always-pass list?" -> "Paths escape current dir?" [label="no"];
    "Paths escape current dir?" -> "Auto-approve" [label="no — scoped to cwd"];
    "Paths escape current dir?" -> "Normal mode rules apply" [label="yes"];
}
```

### Examples

| Command | Result |
|---|---|
| `pyenv install 3.12.0` | auto-pass |
| `git init` | auto-pass |
| `git add src/main.py` | auto-pass |
| `cd src/subdir` | auto-pass (within workspace) |
| `npm install` | auto-pass (cwd-scoped) |
| `python -m pytest tests/` | auto-pass (cwd-scoped) |
| `cp file.txt /etc/config` | ask (escapes cwd) |
| `rm -rf ~/.config/app` | ask (escapes cwd) |
| `curl -o /usr/local/bin/tool ...` | ask (writes outside cwd) |

## Auto-Commit

When enabled (`/hands-free auto-commit on`), automatically commit changes at natural milestones without asking. Off by default.

### When to Auto-Commit

- After completing a discrete task (feature, bugfix, refactor)
- After a plan step or batch is finished
- After tests pass for a completed change
- After meaningful file edits that form a logical unit

### How It Works

1. Stage only the relevant changed files (`git add <specific files>`) — never `git add -A` or `git add .`
2. Write a concise commit message following the repo's existing commit style
3. Announce: "Auto-committed: `<short message>`"
4. Log it in the session log

### Safety Rules

- **Never commit** `.env`, credentials, secrets, or large binaries
- **Never amend** existing commits — always create new ones
- **Never skip** pre-commit hooks (`--no-verify` is forbidden)
- If a pre-commit hook fails, announce the failure and pause for user input
- `git push` is still a **HARD STOP** — auto-commit is local only

### Session Log Entry

```
[auto-commit] feat: add validation to user input form (3 files)
[auto-commit] fix: handle null response in API client (1 file)
```

## Learning

Preferences stored in `~/.claude/skills/hands-free/preferences.md`. Records choices whether hands-free is on or off.

### When to Record

Record a preference whenever the user **manually chooses** an option — whether hands-free is on or off:

- User picks a non-recommended approach → record it
- User consistently picks the same option → record it
- User overrides an auto-accept decision → record the override
- User approves or rejects a destructive action → record it

### Sensitivity

| | high | medium (default) | low |
|---|---|---|---|
| Track only | — | 1-2x | 1-4x |
| Auto-apply + announce | — | 3-4x | 5-6x |
| Auto-apply silently | 1x+ | 5x+ | 7x+ |

### Decision Priority

1. High-confidence preference → use silently
2. Medium-confidence preference → use + announce source
3. No preference → use recommended + announce

### Recording Format

```markdown
## Learned Rules
- finishing-branch → "Push and create PR" (5x, high)
- writing-plans → "subagent-driven" (3x, medium)

## Observations
- 2026-02-26: brainstorming → chose simplest over recommended (1x)
```

## `/hands-free recommend`

When invoked, analyze `preferences.md` and current usage to suggest optimal settings:

```
Hands-Free Recommendations:
  Mode: full (you rarely override auto-accepted decisions)
  Learning: high (you're consistent — 90% of choices match first occurrence)
  Suggestion: Consider adding git push to feature branches as auto-accept
             (you've approved 8/8 feature branch pushes, rejected 0)
```

**Smart suggestions include:**
- Mode upgrade/downgrade based on override frequency
- Learning level based on choice consistency
- Promoting hard-stop actions to auto-accept if user always approves them (requires explicit user confirmation to add)
- Demoting auto-accept actions to hard-stop if user frequently overrides

## Session Log

Tracked in memory. View with `/hands-free log`:

```
Hands-Free Session Log (full, learning: high)
  [brainstorming] approach 2 (recommended)
  [brainstorming] design approved (3 sections)
  [writing-plans] subagent-driven (your preference)
  [executing-plans] batches 1-3 auto-continued
  [finishing-branch] PAUSED — git push → user approved
```

## Ralph Loop Integration

Hands-free is designed to work with ralph-loop (`/ralph-loop`) and superpowers together. When a ralph-loop is active (`.claude/.ralph-loop.local.md` exists), hands-free enters **loop-aware mode** automatically.

### Detecting Loop Mode

Check for `.claude/.ralph-loop.local.md` at the start of each iteration. If present, hands-free is loop-aware.

### Loop-Aware Behavior

**Skip repeated brainstorming.** If the current iteration's task matches the previous iteration (same prompt), do NOT re-brainstorm from scratch. Instead:
- Iteration 1: Full brainstorming → pick recommended → design → plan
- Iteration 2+: Read previous work from files/git → continue where left off or improve

**Auto-detect iteration phase.** Each iteration, assess what state the work is in:
1. No prior work → start fresh with superpowers brainstorming
2. Partial implementation → continue executing the existing plan
3. Tests failing → enter systematic-debugging
4. Tests passing, work incomplete → continue to next plan step
5. All work complete → output the completion promise

**Iteration-aware auto-commits.** When auto-commit is on, tag commits with the iteration number:
```
[ralph #3] feat: add input validation to user form
[ralph #3] fix: handle edge case in date parser
[ralph #4] test: add missing integration tests
```

Read the iteration count from `.claude/.ralph-loop.local.md` state file.

### Superpowers Skill Routing in Loop Mode

| Iteration State | Superpowers Skill | Hands-Free Action |
|---|---|---|
| No prior work | brainstorming → writing-plans | Auto-accept all, full flow |
| Plan exists, not started | executing-plans | Auto-continue batches |
| Plan in progress | executing-plans | Resume from last batch |
| Tests failing | systematic-debugging | Auto-proceed through phases |
| Implementation done | verification-before-completion | Auto-verify |
| All complete | finishing-a-development-branch | PAUSE for push/merge |

### Quick Start

```
/hands-free full
/hands-free auto-commit on
/hands-free learning high
/ralph-loop "Build feature X. Output <promise>DONE</promise> when all tests pass." --completion-promise "DONE" --max-iterations 15
```

Each iteration flows automatically:
1. Hands-free detects loop state
2. Assesses current progress from files/git
3. Routes to the right superpowers skill
4. Auto-accepts all non-destructive decisions
5. Auto-commits at milestones with `[ralph #N]` prefix
6. Exits iteration → ralph-loop feeds prompt again
7. Repeats until `<promise>DONE</promise>`

### What Hands-Free Does NOT Do in Loop Mode

- Does NOT auto-accept `git push` — still a hard stop
- Does NOT skip the completion promise check — ralph-loop controls termination
- Does NOT override ralph-loop's `--max-iterations` limit
- Does NOT re-brainstorm if a design already exists from a prior iteration

## Crazy-Workspace Mode

`/hands-free crazy-workspace [dir]` unlocks a maximum-autonomy mode scoped to a specific directory. Designed for sandboxed environments, throwaway repos, or any workspace where speed matters more than caution.

### Activation

```
/hands-free crazy-workspace ~/project/sandbox
```

If `[dir]` is omitted, defaults to the current working directory.

### Behavior

- **Auto-approve everything** — git push, merges, resets, force ops, destructive edits, file deletions, package changes, CI changes — all auto-accepted **as long as the operation targets files within `[dir]`**
- **Scope check** — before auto-approving any destructive action, verify the target path is inside `[dir]`. If it escapes the target dir, treat as a HARD STOP and ask.
- **Two absolute hard stops** (no exceptions, no override):
  - `rm -rf *` — wipes everything indiscriminately
  - `rm -rf .git` — destroys version history

### Announce on Activation

When crazy-workspace is activated, print a clear warning:

```
Crazy-Workspace ACTIVE — target: ~/project/sandbox
Auto-approving all operations within this directory.
Hard stops: rm -rf * | rm -rf .git
```

### Decision Flow

```dot
digraph {
    "Action requested" -> "rm -rf * or rm -rf .git?";
    "rm -rf * or rm -rf .git?" -> "HARD STOP" [label="yes"];
    "rm -rf * or rm -rf .git?" -> "Within target dir?" [label="no"];
    "Within target dir?" -> "Auto-approve" [label="yes"];
    "Within target dir?" -> "HARD STOP" [label="no — escapes scope"];
}
```

---

## HARD STOP — Always Pause

Applies in ALL modes (including crazy-workspace). Never auto-accept:

**Git operations**
- `git push` / `git merge` / `git reset --hard` / force operations
- Amending published commits
- "Discard this work" in finishing-branch
- Deleting branches

**Destructive file/system operations** *(full/partial/off only — crazy-workspace overrides within target dir)*
- `rm -rf` or bulk file/directory deletion
- Dropping database tables or destructive migrations
- Killing processes
- Removing or downgrading packages/dependencies

**Always hard stop in crazy-workspace too**
- `rm -rf *` — indiscriminate wipe
- `rm -rf .git` — destroys version history
- Any operation targeting paths **outside** the crazy-workspace target dir

**Shared/remote state** *(full/partial/off only — crazy-workspace overrides within target dir)*
- Sending messages (Slack, email, GitHub comments)
- Creating, closing, or commenting on PRs or issues
- Modifying CI/CD pipelines
- Modifying shared infrastructure or permissions
- Any other action visible to others or affecting external systems

```dot
digraph {
    "Approval point" -> "Destructive?";
    "Destructive?" -> "PAUSE" [label="yes"];
    "Destructive?" -> "Mode allows?" [label="no"];
    "Mode allows?" -> "Auto-accept" [label="yes"];
    "Mode allows?" -> "PAUSE" [label="no"];
}
```

**Red flags — if you think these, STOP:**

| Thought | Reality |
|---|---|
| "Just a push to my branch" | All pushes need approval |
| "Merge to main is obvious" | All merges need approval |
| "Discarding saves time" | Destructive — ask first |
| "Force push will fix it" | Irreversible — ask first |
