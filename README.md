# Hands-Free Skill

A Claude Code skill that auto-accepts recommended options from any skill workflow, giving you a hands-off experience. Works with [superpowers](https://github.com/anthropics/claude-code-plugins) and any custom skill that presents choices.

## What it does

- **Auto-accepts** recommended options at approval points (brainstorming, design, execution, checkpoints)
- **Auto-commits** changes at natural milestones with clean git history
- **Pauses** for destructive/irreversible actions (git push, merge, discard, force operations, rm -rf, CI/CD changes, etc.)
- **Blocks** pipe-to-shell (`curl | bash`), privilege escalation (`chmod 777`), and secrets in commits — always, in every mode
- **Review checkpoints** — structured pause-and-summarize before costly phase transitions (optional, or always-on in `partial`)
- **Explains** decisions with `/hands-free explain` — trace why a specific auto-accept happened
- **Learns** your preferences over time — tracks choices, builds confidence, adapts to you
- **Ralph-loop integration** — works with ralph-loop + superpowers for fully autonomous iterative development
- **Four modes** — `full`, `partial`, `off`, and `crazy-workspace` for different levels of autonomy

## Install

Clone and symlink (recommended — stays in sync with updates):

```bash
git clone git@github.com:joe888777/autopilot-skill.git ~/hands-free-skill
ln -s ~/hands-free-skill ~/.claude/skills/hands-free
```

Or copy:

```bash
git clone git@github.com:joe888777/autopilot-skill.git
cp -r autopilot-skill ~/.claude/skills/hands-free
```

## Usage

```
/hands-free              # Full auto-accept (default)
/hands-free full         # Same as above
/hands-free partial      # Auto-accept design, pause at execution
/hands-free off          # Back to normal
/hands-free crazy-workspace       # Max autonomy within ./
/hands-free auto-commit on        # Auto-commit at natural milestones
/hands-free auto-commit off       # Disable auto-commit (default)
/hands-free review-checkpoints on # Pause at phase transitions for review
/hands-free learning <h/m/l>      # Set learning sensitivity (high/medium/low)
/hands-free dry-run               # Preview what would be auto-accepted
/hands-free pause                 # Temporarily suspend auto-accept (mode preserved)
/hands-free resume                # Resume auto-accept after pause
/hands-free explain               # Explain why the last auto-accept decision was made
/hands-free reset                 # Clear all learned preferences (requires confirmation)
/hands-free status                # Show current mode + all settings
/hands-free recommend             # Suggest optimal settings
/hands-free log                   # Show session decisions
```

### Modes

| Approval Type | full | partial | off | crazy-workspace |
|---|---|---|---|---|
| Brainstorming / design / phase transitions | auto | auto | ask | auto |
| Execution method | auto | **ask** | ask | auto |
| Batch checkpoints | auto | **ask** | ask | auto |
| Shell cmds scoped to cwd / pyenv / git add | auto | auto | ask | auto |
| Destructive actions (within workspace) | **ask** | **ask** | ask | auto |
| Review checkpoint (before execution) | skip | **HARD STOP** | **HARD STOP** | skip |
| Review checkpoint (before push/merge) | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| Git push | **HARD STOP** | **HARD STOP** | **HARD STOP** | auto |
| `curl \| bash` / pipe-to-shell | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| `chmod 777` / privilege escalation | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| Secrets in staged files | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| `rm -rf *` | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| `rm -rf .git` | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |

### Example

Without hands-free:
```
Claude: "I see 3 approaches. Which do you prefer?"
You: "2"
Claude: "Design section 1: ... Does this look right?"
You: "yes"
Claude: "Subagent-driven or parallel session?"
You: "subagent"
```

With `/hands-free`:
```
Claude: "Going with approach 2 (recommended) — best balance of simplicity and flexibility."
Claude: "Design approved. Moving to implementation plan."
Claude: "Going with subagent-driven (recommended for this session)."
```

### Checking what's active

```
/hands-free status

Hands-Free Status
  Mode:                 full
  Learning:             high
  Auto-commit:          on
  Review checkpoints:   off
  Paused:               no
  Loop-aware:           no

  Session decisions:    7 auto-accepted, 1 paused
  Preferences loaded:   2 rules (1 high, 1 medium)

  Universal hard stops: curl|bash, chmod 777, secrets-in-commit, rm -rf *, rm -rf .git
  Mode hard stops:      git push, git merge, git reset --hard (paused in this mode)
```

### Previewing before enabling

```
/hands-free dry-run

Hands-Free Dry Run — current mode: off, learning: medium

Would auto-accept (if full mode enabled):
  Brainstorming approach selection    yes
  Design approval                     yes
  Shell commands scoped to cwd        yes

Would PAUSE for (always, all modes):
  git push                            HARD STOP
  curl | bash                         HARD STOP
  chmod 777                           HARD STOP
  Secrets in staged files             HARD STOP
  rm -rf *                            HARD STOP

To enable: /hands-free full
```

## Ralph Loop + Superpowers Example

Hands-free combines with [ralph-loop](https://ghuntley.com/ralph/) and superpowers for fully autonomous iterative development. Type 4 lines, walk away, come back to working code.

### Setup

```
/hands-free full
/hands-free auto-commit on
/hands-free learning high
/ralph-loop "Build a REST API for a todo app with Express.js. Include CRUD endpoints, input validation, error handling, and tests. Output <promise>DONE</promise> when all tests pass." --completion-promise "DONE" --max-iterations 10
```

### What happens

**Iteration 1 — Design & scaffold**
```
[hands-free] Loop detected — iteration #1, no prior work → full superpowers flow
[hands-free] brainstorming → Going with approach 2 (recommended) — Express + Zod + Vitest
[hands-free] design approved (3 sections)
[hands-free] writing-plans → subagent-driven (recommended)
[auto-commit] [ralph #1] feat: scaffold Express app with CRUD routes (4 files)
[auto-commit] [ralph #1] feat: add Zod validation and error handler (3 files)
[auto-commit] [ralph #1] test: add CRUD endpoint tests (2 files)

Running tests... 3 passed, 2 failed → no completion promise, exiting iteration
```

**Iteration 2 — Fix failures**
```
[hands-free] Loop detected — iteration #2, tests failing → systematic-debugging
[auto-commit] [ralph #2] fix: handle null request body and missing todo 404 (2 files)

Running tests... 5 passed, 0 failed → edge cases not covered, no promise yet
```

**Iteration 3 — Add edge cases**
```
[hands-free] Loop detected — iteration #3, tests passing, work incomplete → continue plan
[auto-commit] [ralph #3] test: add edge case validation tests (1 file)

Running tests... 9 passed, 1 failed
```

**Iteration 4 — Final fix**
```
[hands-free] Loop detected — iteration #4, tests failing → systematic-debugging
[auto-commit] [ralph #4] fix: add duplicate title check in create endpoint (1 file)

Running tests... 10 passed, 0 failed
<promise>DONE</promise>
```

### Result

```
/hands-free log

Hands-Free Session Log (full, learning: high, ralph-loop: 4 iterations)
  [brainstorming] approach 2 — Express + Zod + Vitest
  [brainstorming] design approved (3 sections)
  [writing-plans] subagent-driven (recommended)
  [executing-plans] batches 1-3 auto-continued
  [auto-commit] [ralph #1] scaffold, validation, tests (3 commits)
  [systematic-debugging] 2 failures → fixed
  [auto-commit] [ralph #2] null body + 404 fix
  [auto-commit] [ralph #3] edge case tests
  [systematic-debugging] 1 failure → fixed
  [auto-commit] [ralph #4] duplicate check
  COMPLETED — 4 iterations, 7 commits, 10 tests passing
```

Git log:
```
* [ralph #4] fix: add duplicate title check in create endpoint
* [ralph #3] test: add edge case validation tests
* [ralph #2] fix: handle null request body and missing todo 404
* [ralph #1] test: add CRUD endpoint tests
* [ralph #1] feat: add Zod validation and error handler
* [ralph #1] feat: scaffold Express app with CRUD routes
```

## How learning works

Hands-free records your choices in `preferences.md` whether it's on or off.

| Occurrences | Confidence | Behavior |
|---|---|---|
| 1-2x | low | Tracked, not auto-applied |
| 3-4x | medium | Auto-applied with announcement |
| 5x+ | high | Auto-applied silently |

Over time, hands-free picks **what you would pick**, not just the default recommendation.

## Session log

Run `/hands-free log` anytime to see a summary of all auto-accepted decisions in the current session.

## Security

Hands-free enforces **5 universal hard stops** in ALL modes, including `crazy-workspace`. These cannot be overridden, promoted to auto-accept, or disabled:

| Pattern | Why |
|---|---|
| `curl \| bash`, `wget \| sh`, `eval $(curl ...)` | Remote code execution — arbitrary code from the internet |
| `chmod 777`, `chmod a+rwx` | World-writable permissions — any user can modify the file |
| Secrets in staged files | Prevent accidentally committing API keys, private keys, tokens |
| `rm -rf *` | Indiscriminate wipe — deletes everything in scope |
| `rm -rf .git` | Destroys version history — unrecoverable without a backup |

All other hard stops (git push, merge, destructive ops) apply in `full`/`partial`/`off` modes but are auto-approved in `crazy-workspace` within `./`.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- Works with [Superpowers](https://github.com/anthropics/claude-code-plugins), custom skills, or any workflow with approval points

## Contributing

PRs welcome. If you find an approval point that hands-free doesn't handle well, open an issue.

## License

MIT
