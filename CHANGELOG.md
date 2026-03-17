# Changelog

## [Unreleased] — 2026-03-17 (iteration 2)

### Added

- **`/hands-free reset`**: new command to clear all learned preferences; always prompts for confirmation, never auto-accepted
- **Mode transitions**: defined behavior for switching modes mid-session (immediate effect, no retroactive changes); `partial` mode automatically enables review-checkpoints
- **Conflict resolution**: priority rules when two skills present simultaneous approval points (hard stop wins, more restrictive wins, preferences override both)
- **Session log events**: added `[review-checkpoint]`, `[auto-commit]`, `[verification]`, `[hard-stop]`, `[user-override]` event types to log format
- **Recommended Setup table**: expanded from single recommendation to use-case table covering 5 scenarios

### Fixed

- **Crazy-workspace activation announcement**: updated to list all 6 hard stops (was incorrectly listing only `rm -rf *` and `rm -rf .git`)
- **Crazy-workspace "Two absolute hard stops"**: updated description to "Absolute hard stops (no exceptions)" to reflect the 5 universal hard stops now enforced
- **Crazy-workspace decision flowchart**: updated to check pipe-to-shell, chmod 777, and secrets in addition to `rm -rf *` and `rm -rf .git`
- **HARD STOP section**: clarified two-tier structure (universal vs. standard); git push and merge now correctly annotated as crazy-workspace overrideable within `./`

### Changed

- README "What it does": added pipe-to-shell/secrets blocking and review checkpoints to feature list; updated "Three modes" to "Four modes" (adds crazy-workspace)

## [Unreleased] — 2026-03-17 (iteration 1)

### Added

- **Security — pipe-to-shell hard stop**: `curl | bash`, `wget | sh`, `eval $(curl ...)`, and `bash <(curl ...)` are now HARD STOPs in all modes including crazy-workspace
- **Security — privilege escalation hard stop**: `chmod 777`, `chmod -R 777`, and `sudo` writing to system paths are now HARD STOPs in all modes including crazy-workspace
- **Security — secrets detection in auto-commit**: pre-commit scan checks filename patterns (`.env`, `*.pem`, `*.key`, `*.secret`, etc.) and content signals (`sk-`, `ghp_`, `AKIA`, `-----BEGIN RSA`, `password=`, `token=`, etc.) before every auto-commit; blocks with announcement; applies in all modes including crazy-workspace
- **Security — path traversal clarification**: symlink escapes, `..`-normalized paths, and shell variable expansions (`$HOME`, `~`, `$XDG_*`) that exit cwd now block auto-pass in shell auto-pass rules
- **Automation — `/hands-free dry-run`**: preview what would be auto-accepted with current mode and learned preferences, without changing state or enabling hands-free
- **Automation — iteration warnings**: prints warning at 3 and 2 remaining iterations; mandatory PAUSE at 1 remaining iteration before ralph-loop terminates
- **Automation — git-log-based loop state detection**: replaces heuristic prose with a concrete algorithm using `git log --oneline -20` and `git status --short`; five-state decision table maps git state to the correct superpowers skill
- **Review — review checkpoint system**: structured pause-and-summarize before phase transitions; two mandatory checkpoints (before execution starts, before push/merge); optional checkpoints at other phase transitions via `/hands-free review-checkpoints on/off`
- **UX — `/hands-free status`**: now fully defined with concise output showing mode, learning, auto-commit, review checkpoints, loop state, session decisions, and active hard stops
- **UX — `/hands-free dry-run` command**: added to commands table and implemented
- **UX — `/hands-free review-checkpoints` command**: added to commands table

### Changed

- Mode behavior table: added `curl | bash`, `chmod 777`, and review checkpoint rows (now 14 rows total, up from 10)
- Auto-commit safety rules: expanded from a single bullet to a full secrets detection procedure with filename patterns and content signals
- Shell auto-pass rules: added symlink escapes, normalized `..` paths, and `$HOME`/`~`/`$XDG_*` expansions as path-escape conditions
- Loop state detection: replaced 5 heuristic prose bullets with a concrete git-log decision table
- Hard stop section: added "Remote code execution patterns" and "Privilege escalation" subsections before the existing destructive operations list
- Red flags table: added 4 new entries for pipe-to-shell, chmod 777, secrets in comments, and symlink escapes
- crazy-workspace hard stops: now explicitly lists pipe-to-shell, privilege escalation, and secrets detection alongside `rm -rf *` and `rm -rf .git`
- README mode table: expanded from 5 rows to 13 rows; added crazy-workspace column; includes all new hard stop patterns
- README commands section: now lists all 13 commands including dry-run, review-checkpoints, crazy-workspace, auto-commit, status, and recommend

## [Unreleased] — 2026-03-16

### Changed

- **crazy-workspace mode**: Simplified behavior description — auto-approve all operations within `./` without the redundant qualifier. Hard stops remain: `rm -rf *` and `rm -rf .git`.
