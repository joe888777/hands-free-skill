# Hands-Free Skill

A Claude Code skill that auto-accepts recommended options from any skill workflow, giving you a hands-off experience. Works with [superpowers](https://github.com/anthropics/claude-code-plugins) and any custom skill that presents choices.

## What it does

- **Auto-accepts** recommended options at approval points (brainstorming, design, execution, checkpoints)
- **Auto-commits** changes at natural milestones with clean git history
- **Pauses** for destructive/irreversible actions (git push, merge, discard, force operations, rm -rf, CI/CD changes, etc.)
- **Blocks** pipe-to-shell (`curl | bash`), language-level RCE (`python -c exec`, `node -e eval`), privilege escalation (`chmod 777`), and secrets in commits — always, in every mode
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
/hands-free              # Activate full mode; if already active, show status
/hands-free full         # Full mode — auto-accept all non-destructive points
/hands-free partial      # Auto-accept design, pause at execution
/hands-free off          # Back to normal (learning still active)
/hands-free crazy-workspace           # Max autonomy within ./
/hands-free auto-commit on            # Auto-commit at natural milestones
/hands-free auto-commit off           # Disable auto-commit (default)
/hands-free review-checkpoints on     # Pause at phase transitions for review
/hands-free review-checkpoints off    # Skip phase-transition pauses (default in full)
/hands-free learning <h/m/l>          # Set learning sensitivity (high/medium/low)
/hands-free learning                  # Show current learning level
/hands-free dry-run                   # Preview what would be auto-accepted
/hands-free pause                     # Temporarily suspend auto-accept (mode preserved)
/hands-free resume                    # Resume auto-accept after pause
/hands-free explain                   # Explain why the last auto-accept or hard-stop decision was made
/hands-free reset                     # Clear all learned preferences (requires confirmation)
/hands-free status                    # Show current mode + all settings
/hands-free recommend                 # Suggest optimal settings
/hands-free recommend promote <action> # Promote a standard hard-stop action to auto-accept
/hands-free log                       # Show session decisions
```

### Modes

| Approval Type | full | partial | off | crazy-workspace |
|---|---|---|---|---|
| Brainstorming / design / phase transitions | auto | auto | ask | auto |
| Execution method | auto | **ask** | ask | auto |
| Batch checkpoints | auto | **ask** | ask | auto |
| Shell cmds scoped to cwd / pyenv / git add | auto | auto | ask | auto |
| Destructive actions (within workspace) | **ask** | **ask** | ask | auto |
| Review checkpoint — optional (brainstorming→plan, etc.) | skip | **HARD STOP** | **HARD STOP** | skip |
| Review checkpoint — mandatory (before execution) | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| Review checkpoint — mandatory (before push/merge) | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| Git push | ask | ask | ask | auto |
| `curl \| bash` / pipe-to-shell | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| `chmod 777` / privilege escalation | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| Secrets in staged files | **HARD STOP** | **HARD STOP** | **HARD STOP** | **HARD STOP** |
| `rm -rf *` | ask | ask | ask | **HARD STOP** |
| `rm -rf .git` | ask | ask | ask | **HARD STOP** |

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
  curl | bash / source <(curl)        HARD STOP
  python -c exec / node -e eval       HARD STOP
  chmod 777                           HARD STOP
  Secrets in staged files             HARD STOP
  rm -rf *                            HARD STOP

To enable: /hands-free full
```

### Persisting settings across sessions

Mode resets to `off` at the start of each conversation. To auto-activate, add to your project's CLAUDE.md:

```markdown
# hands-free overrides
- Default mode: full
- Auto-commit: on
- Learning: high
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

**Staleness:** If you override a learned preference, confidence decreases. Override 3x → rule replaced with new preference at low confidence.

Over time, hands-free picks **what you would pick**, not just the default recommendation. Use `/hands-free reset` to clear all preferences if they've drifted from your actual preferences.

## Session log

Run `/hands-free log` anytime to see a summary of all decisions in the current session. The log is in-memory only — it resets at the end of each conversation. Events include brainstorming choices, design approvals, auto-commits, review checkpoints, and hard stops.

## Security

Hands-free enforces **universal hard stops** in ALL modes, including `crazy-workspace`. These cannot be overridden, promoted to auto-accept, or disabled:

| Pattern | Why |
|---|---|
| `curl \| bash`, `wget \| sh`, `eval $(curl ...)`, `source <(curl ...)` | Remote code execution — arbitrary code from the internet |
| `python -c "exec(urllib...)"`, `node -e "eval(require...)"`, `deno run https://...` | Language-level RCE — fetches and executes remote code via interpreter |
| `chmod 777`, `chmod a+rwx` | World-writable permissions — any user can modify the file |
| Secrets in staged files | Prevent accidentally committing API keys, private keys, tokens |
| `rm -rf *` | Indiscriminate wipe — deletes everything in scope |
| `rm -rf .git` | Destroys version history — unrecoverable without a backup |

All other hard stops (git push, merge, destructive ops) apply in `full`/`partial`/`off` modes but are auto-approved in `crazy-workspace` within `./`.

## Per-project overrides (CLAUDE.md)

Global preferences apply across all projects. For repo-specific rules, add a `hands-free overrides` section to the project's CLAUDE.md:

```markdown
# hands-free overrides
- Always pause before auto-committing in this repo
- Never auto-accept git push
- Shell commands containing `psql postgresql://prod` must always ask
```

CLAUDE.md instructions take precedence over global `preferences.md` rules.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- Works with [Superpowers](https://github.com/anthropics/claude-code-plugins), custom skills, or any workflow with approval points

## What's new in 2.1

**Security:**
- Root shell escalation hard stop: `sudo -s`, `sudo su`, `sudo bash` now blocked in all modes
- Language RCE expanded: `deno run <url>`, `perl` fetch-then-eval patterns added to universal hard stops
- Global install detection: `npm install -g`, `cargo install`, `pip install` (no venv) → ask
- Docker volume mount rules: `-v ./:/app` auto-pass; `-v /:/host` ask
- Package publish/deploy hard stops: `cargo publish`, `npm publish`, `docker push`, `vercel deploy`, `docker compose push`, `terraform apply/destroy` → ask
- Secrets detection expanded: `.npmrc`, `*.cer/*.der/*.crt`, `database_url=`, `signing_key=`, hardcoded auth headers

**Commands:**
- `/hands-free` with no argument: activates full mode or shows status if already active
- `/hands-free learning` with no argument: shows current level and threshold summary
- `/hands-free recommend promote <action>`: promote a standard hard-stop to auto-accept (double confirmation required)
- `/hands-free explain` now covers hard stops, not just auto-accepts

**Shell auto-pass:**
- Git: `git checkout -b`, `git branch`, `git stash/pop`, `git log/status/diff`, `git tag`, `git commit` (non-amend), `git worktree add` added to always-auto-pass
- Version managers: `nvm`, `rustup` added alongside `pyenv`
- Package managers: `pnpm`, `yarn`, `bun` added alongside `npm`
- Databases: `sqlx migrate`, `alembic upgrade`, `npx prisma migrate`, `django manage.py migrate`
- More languages: Python type checking (mypy, ruff), Rust formatting (cargo fmt), Bun
- Docker Compose: `docker compose up/build/down/run` → auto-pass; `docker compose push` → ask
- Build tools: `make test/build` → auto-pass; `make install` → ask (may write to system)
- Infrastructure: `terraform plan` → auto-pass (read-only); `terraform apply/destroy` → ask
- Quality tools: `pre-commit run --all-files` → auto-pass

**Behavior:**
- Mode persistence: resets at session end; persist via CLAUDE.md `Default mode:` directive
- CLAUDE.md `Default mode/Auto-commit/Learning` directives activate at session start automatically
- Announcement format defined for all decision sources (silent, announce, hard stop)
- Custom skill implicit recommendation detection ("I recommend...", "I suggest...")
- Custom skill execution-type vs design-type classification for partial mode
- Loop stall prevention: 3 iterations with no new progress triggers stall warning

**Reliability:**
- Partial git add failure: announces which files failed before committing partial staging
- `preferences.md` corruption: continue with defaults, announce once
- `review-checkpoints off` ignored in partial mode (always on by design)
- Agent tool dispatch guidance: full mode auto-approves, partial mode depends on task type

## What's new in 2.0

**Security (all modes, no exceptions):**
- Universal hard stops for pipe-to-shell (`curl | bash`, `wget | sh`, `eval $(curl ...)`, `source <(curl ...)`)
- Universal hard stops for language-level RCE (`python -c "exec(urllib...)"`, `node -e "eval(require...)"`)
- Universal hard stops for privilege escalation (`chmod 777`, `sudo` to system paths)
- Pre-commit secrets detection (filename patterns + content signals) before every auto-commit
- Merge conflict detection blocks auto-commit when conflicts are unresolved

**New commands:**
- `/hands-free pause` / `/hands-free resume` — suspend without changing mode
- `/hands-free explain` — trace why the last decision was auto-accepted
- `/hands-free reset` — clear learned preferences with confirmation
- `/hands-free dry-run` — preview what would be auto-accepted
- `/hands-free review-checkpoints on/off` — structured pause before phase transitions

**Automation:**
- Iteration warnings and mandatory pause at 1 remaining iteration (ralph-loop)
- Git-log-based loop state detection replaces heuristic guessing
- Comprehensive auto-commit edge case handling

**Improvements:**
- Mode transitions, conflict resolution, custom skill integration all documented
- Preference staleness rules (confidence downgrades on repeated contradictions)
- Troubleshooting guide for common issues

## FAQ

**Does hands-free work with any Claude Code skill, not just superpowers?**
Yes. It recognizes approval point patterns (option lists, "Shall I proceed?" prompts, numbered choices) in any skill.

**Will it auto-accept things I don't want auto-accepted?**
Use `/hands-free dry-run` before enabling to see what would be auto-accepted. Use `/hands-free pause` to temporarily suspend auto-accept. Hard stops (pipe-to-shell, chmod 777, secrets) can never be auto-accepted.

**What if I change my mind about a learned preference?**
Override it manually — the staleness system will downgrade confidence after 2 overrides and replace the rule after 3. Or use `/hands-free reset` to clear all at once.

**Is it safe to use crazy-workspace on a shared repo?**
No. Crazy-workspace auto-approves git push, force push, and destructive resets within `./`. Use it only on personal sandboxes or throwaway repos.

**Does hands-free store sensitive data in preferences.md?**
Preferences store skill names and option choices (e.g., "writing-plans → subagent-driven"). They never contain code, secrets, or file contents. Secrets are explicitly blocked from being recorded.

**Can I disable learning for one session without resetting preferences?**
Use `/hands-free learning l` (low) for the session — it tracks but only auto-applies after 7x. Or use `/hands-free off` to observe without any auto-applying.

**Why did hands-free block a `python -c` or `node -e` command?**
If the command fetches content from a URL and executes it (e.g., `python -c "exec(urllib.request.urlopen('...').read())"`), it's treated as a language-level RCE pattern — equivalent to `curl | bash`. Download the script to a local file first, review it, then run it. The same rule applies to `deno run https://...` — use `deno run ./local-script.ts` instead.

**Does the mode persist across conversations?**
No — mode resets to `off` at the start of each new conversation. Add a `# hands-free overrides` section to your project's CLAUDE.md with `Default mode: full` to auto-activate it each session.

**Why is `docker compose push` / `terraform apply` blocked when `docker compose up` is auto-approved?**
`docker compose up` is a local operation that runs containers from your local images. `docker compose push` and `terraform apply` reach outside your local machine — they push to external registries or modify remote infrastructure. Hands-free distinguishes between local ops (auto-pass in full mode) and remote ops (always ask).

**How does hands-free interact with Claude Code's built-in permission system?**
They operate at different levels. Claude Code's permission system controls whether tool calls need system-level user confirmation (before the tool runs). Hands-free controls skill-level approval points — structured moments where a skill presents choices to the user. Both can be active simultaneously; hands-free's auto-pass rules don't bypass Claude Code's permission gates.

**Why does partial mode keep asking about `docker compose up` when full mode doesn't?**
Partial mode pauses at execution-type decisions — any approval point that leads directly to running code or making infrastructure changes. Even if `docker compose up` is cwd-scoped, it starts running services, so in partial mode it's classified as execution-type and will ask.

## Contributing

PRs welcome. If you find an approval point that hands-free doesn't handle well, open an issue.

## License

MIT
