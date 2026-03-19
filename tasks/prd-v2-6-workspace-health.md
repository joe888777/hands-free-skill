[PRD]
# PRD: Hands-Free v2.6.0 — Workspace Health & Pre-flight Checks

## Overview

Loop iterations occasionally fail mid-session due to dirty git state (unstaged changes blocking `git pull --rebase`, detached HEAD, merge conflicts). This release adds a `/hands-free health` command that reports workspace status in one summary, pre-flight checks that detect and auto-heal common git state issues at loop iteration start, and an auto-stash/unstash pattern that prevents `git pull --rebase` from failing due to local modifications.

## Goals

- Eliminate `git pull --rebase` failures caused by unstaged or uncommitted changes in loop iterations
- Provide a single `/hands-free health` command that surfaces git state, build state, test state, and security posture
- Auto-heal the three most common workspace issues (uncommitted changes, detached HEAD detection, merge conflict detection) before attempting a pull
- Block loop execution gracefully when the workspace is in an unhealthy state that cannot be auto-healed, rather than silently continuing and failing mid-iteration

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.6.0`
- `grep "\[2.6.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Add `/hands-free health` command

**Description:** As a skill user, I want a single command that reports the complete workspace health so I can diagnose issues at a glance.

**Acceptance Criteria:**
- [ ] `/hands-free health` command added to the Commands table in SKILL.md
- [ ] Command outputs a structured health report with four sections: Git State, Build State, Test State, Security Posture
- [ ] Git State section reports: current branch, clean/dirty status, ahead/behind counts, merge-conflict presence, detached-HEAD status
- [ ] Build State section reports: project type detected (Rust/Python/JS/unknown), last build result (pass/fail/unknown)
- [ ] Test State section reports: last test run result (pass/fail/N tests/unknown)
- [ ] Security Posture section reports: grade from `.claude/security-posture.json` if present, else "not yet scanned"
- [ ] Overall health verdict: `HEALTHY` / `DEGRADED` (fixable issues) / `BLOCKED` (requires manual intervention)
- [ ] Example output block included in SKILL.md showing the full formatted report
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add pre-flight check spec to loop-aware behavior

**Description:** As a skill user running ralph-loop, I want hands-free to automatically verify and fix workspace state at the start of each iteration so that git operations don't fail mid-session.

**Acceptance Criteria:**
- [ ] New "Pre-flight Check" sub-section added under `## Ralph Loop Integration` → `### Loop-Aware Behavior`
- [ ] Pre-flight runs BEFORE the build state health check in the existing algorithm
- [ ] Pre-flight checks defined in order: (1) detect merge conflicts → BLOCKED if present, (2) detect detached HEAD → BLOCKED if present, (3) detect staged uncommitted changes → auto-commit or warn, (4) detect unstaged modifications → auto-stash
- [ ] Pre-flight outcome table included: condition → action → announce message
- [ ] BLOCKED conditions cause hands-free to output `[hands-free] Pre-flight BLOCKED — [reason]. Resolve manually before continuing.` and halt the iteration
- [ ] DEGRADED conditions (auto-healed) cause hands-free to announce what was fixed and continue
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Add auto-stash/unstash pattern for clean pulls

**Description:** As a skill user, I want hands-free to automatically stash local changes before `git pull --rebase` and restore them after so the pull never fails due to local modifications.

**Acceptance Criteria:**
- [ ] Auto-stash/unstash pattern added as a new sub-section `### Auto-Stash Pattern` under `## Ralph Loop Integration`
- [ ] Pattern: if `git status` shows modified tracked files and a pull is needed → `git stash push -m "hands-free pre-pull auto-stash [ralph #N]"` → pull → `git stash pop`
- [ ] If `git stash pop` fails (conflict with pulled changes) → announce `[hands-free] Auto-stash pop failed — stash preserved as stash@{0}. Resolve conflicts manually.` and set iteration state to BLOCKED
- [ ] Stash only runs when branch is behind origin (detected via `git fetch --dry-run` or `git status -sb`)
- [ ] Stash is NOT created if working tree is already clean (no unnecessary stashes)
- [ ] Announce format: `[hands-free] Auto-stashed N file(s) before pull — will restore after rebase`
- [ ] Announce format after restore: `[hands-free] Auto-stash restored (N file(s))`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-004: Bump version to 2.6.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the workspace health feature.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.5.0` to `2.6.0`
- [ ] New `## [2.6.0] — 2026-03-19` section added at the top of CHANGELOG.md (after `# Changelog` heading)
- [ ] CHANGELOG entry summarises: `/hands-free health` command, pre-flight checks, auto-stash/unstash pattern, BLOCKED/DEGRADED state model
- [ ] CHANGELOG entry follows the same format as `## [2.5.0]`
- [ ] `grep "^version:" SKILL.md` shows `2.6.0`
- [ ] `grep "\[2.6.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `/hands-free health` must produce output even when git, build tools, or test runners are not available — show "unknown" rather than erroring
- FR-2: Pre-flight check must run before every loop iteration start, not only when a pull is needed
- FR-3: Auto-stash must use a descriptive message including the ralph iteration number so stashes are identifiable
- FR-4: BLOCKED state must prevent any further work in the iteration; the loop continues to the next iteration
- FR-5: DEGRADED state (issue auto-healed) must not block work — announce and continue
- FR-6: Auto-stash/unstash must be idempotent: if the stash pop succeeds, no stash entry remains; if it fails, the entry is preserved for manual resolution
- FR-7: The health report must be emitted in all modes when `/hands-free health` is explicitly invoked
- FR-8: Version bump to 2.6.0 must appear on its own line in frontmatter (`version: 2.6.0`)

## Non-Goals

- Automatically resolving merge conflicts (only detect and block)
- Modifying the user's git config
- Creating branches or checkouts on behalf of the user
- Running builds or tests as part of the pre-flight (only reporting cached/last-known state)
- Persistent health history across sessions

## Technical Considerations

- SKILL.md is a large markdown file; all edits use the Edit tool with exact string matching
- Each story's changes are in distinct sections: Commands table (US-001), Ralph Loop Integration section (US-002, US-003), frontmatter + CHANGELOG (US-004)
- US-004 depends on US-001 through US-003 (version bump last)
- Auto-stash message format: `"hands-free pre-pull auto-stash [ralph #N]"` where N is the current iteration

## Success Metrics

- `/hands-free health` command documented with complete example output
- Pre-flight check table covers all four condition types
- Auto-stash pattern fully specified with success and failure branches
- Even code-fence count in SKILL.md after all edits
- CHANGELOG [2.6.0] entry present

## Open Questions

- None
[/PRD]
