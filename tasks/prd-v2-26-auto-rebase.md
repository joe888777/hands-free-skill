[PRD]
# PRD: Hands-Free v2.26.0 — Loop Auto-Rebase

## Overview

When a ralph-loop runs on a shared repository, upstream changes may accumulate while the loop is iterating. Without automatic syncing, the loop may diverge from the remote, causing merge conflicts or push failures at the end. This release adds `Loop auto-rebase: on` CLAUDE.md directive, which causes hands-free to run `git pull --rebase` at the start of each iteration before any work begins.

## Goals

- Add `Loop auto-rebase: on/off` CLAUDE.md directive
- When enabled, run `git pull --rebase` at the start of each iteration before any skill routing
- If the rebase fails (conflicts), announce a HARD STOP and pause for user intervention
- Document auto-rebase behavior in Ralph Loop Integration
- Add directive to Available Persistent Settings table
- Bump version to 2.26.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.26.0`
- `grep "\[2.26.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop auto-rebase behavior in Ralph Loop Integration

**Description:** As a skill user, I want hands-free to optionally sync with upstream at each iteration start so the loop stays current with remote changes.

**Acceptance Criteria:**
- [ ] New `### Loop Auto-Rebase` sub-section added under `## Ralph Loop Integration` (after `### Iteration Time Budget`)
- [ ] Section describes: when `Loop auto-rebase: on`, run `git pull --rebase` at the start of each iteration before any brainstorming, planning, or execution begins
- [ ] On success: announce `[hands-free] Auto-rebase: synced with upstream (N commits ahead)` (or "up to date" if no new commits)
- [ ] On rebase conflict: announce `[hands-free] HARD STOP — auto-rebase conflict detected. Resolve conflicts and run /hands-free resume to continue.` and pause for user input
- [ ] Default: off (`Loop auto-rebase:` absent → no auto-rebase)
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop auto-rebase

**Description:** As a skill user, I want to configure loop auto-rebase in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop auto-rebase: on/off` row added to Available Persistent Settings table
- [ ] Effect: when `on`, runs `git pull --rebase` at every iteration start; default: `off`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.26.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the auto-rebase feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.25.0` to `2.26.0`
- [ ] New `## [2.26.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: auto-rebase directive, conflict hard stop, upstream sync announcement
- [ ] `grep "^version:" SKILL.md` shows `2.26.0`
- [ ] `grep "\[2.26.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop auto-rebase: on` runs `git pull --rebase` before any iteration work
- FR-2: On success: print the upstream sync announcement and continue normally
- FR-3: On rebase conflict (non-zero exit): issue HARD STOP, announce conflict, await user resolution
- FR-4: After user resolves conflict and resumes, the iteration continues from the beginning (no partial state)
- FR-5: `Loop auto-rebase: off` (default) — no `git pull --rebase` is ever issued automatically

## Non-Goals

- Handling stash/pop around the rebase (user manages uncommitted work)
- Auto-push after each iteration (separate feature)
- Auto-merge strategy (rebase only)

## Technical Considerations

- US-001 adds `### Loop Auto-Rebase` after `### Iteration Time Budget`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Auto-rebase directive documented with success/failure paths
- Conflict hard stop behavior specified
- Available Persistent Settings table updated
- Even code-fence count, version 2.26.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
