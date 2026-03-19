[PRD]
# PRD: Hands-Free v2.27.0 — Loop Quiet Mode

## Overview

In long ralph-loop runs (50+ iterations), hands-free produces substantial output per iteration: iteration start banners, auto-accept announcements, auto-commit messages, and routing decisions. While useful for short loops, this output can flood the chat history in extended runs and make it difficult to spot important signals. This release adds `Loop quiet: on` mode that suppresses routine per-iteration output, keeping only errors, warnings, hard stops, and the completion promise visible.

## Goals

- Add `Loop quiet: on/off` CLAUDE.md directive
- When enabled, suppress routine per-iteration announcements
- Still show: hard stops, warnings (branch guard, budget exceeded, stall, failure guard), auto-commits (brief), and the completion promise
- Document what is and is not suppressed in quiet mode
- Add directive to Available Persistent Settings table
- Bump version to 2.27.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.27.0`
- `grep "\[2.27.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop quiet mode behavior in Ralph Loop Integration

**Description:** As a skill user, I want a quiet mode for long loops so that routine announcements don't flood the chat.

**Acceptance Criteria:**
- [ ] New `### Loop Quiet Mode` sub-section added under `## Ralph Loop Integration` (after `### Loop Auto-Rebase`)
- [ ] Section defines what is suppressed in quiet mode:
  - Suppressed: iteration start banner (`[hands-free] Iteration N/M — prior work: ...`), auto-accept announcements, routing decisions, review checkpoint skip messages, auto-rebase success messages
  - Still shown: hard stops, warnings (branch guard, budget, stall detection, consecutive failure), auto-commit messages, completion promise output, the 10-iteration cadence summary
- [ ] Note: quiet mode does not affect the session log — all events are still logged to `/hands-free log`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop quiet mode

**Description:** As a skill user, I want to configure quiet mode in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop quiet: on/off` row added to Available Persistent Settings table
- [ ] Effect: when `on`, suppresses routine per-iteration output; default: `off`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.27.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the quiet mode feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.26.0` to `2.27.0`
- [ ] New `## [2.27.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: quiet mode directive, what is suppressed vs still shown, session log unaffected
- [ ] `grep "^version:" SKILL.md` shows `2.27.0`
- [ ] `grep "\[2.27.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop quiet: on` suppresses the iteration start banner and routine auto-accept announcements
- FR-2: Hard stops, warnings, auto-commits, and the completion promise always appear regardless of quiet mode
- FR-3: The 10-iteration cadence summary still fires in quiet mode (it is a compact summary, not a routine announcement)
- FR-4: Session log (`/hands-free log`) captures all events regardless of quiet mode
- FR-5: `Loop quiet: off` (default) — all announcements appear as normal

## Non-Goals

- Silencing hard stops or security events in quiet mode
- Disabling the session log
- Per-event suppression configuration

## Technical Considerations

- US-001 adds `### Loop Quiet Mode` after `### Loop Auto-Rebase` in `## Ralph Loop Integration`
- US-002 adds one row to the Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Quiet mode documented with explicit suppression list
- Session log unaffected by quiet mode
- Available Persistent Settings table updated
- Even code-fence count, version 2.27.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
