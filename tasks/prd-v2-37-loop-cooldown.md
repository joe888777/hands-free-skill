[PRD]
# PRD: Hands-Free v2.37.0 — Loop Cooldown

## Overview

When a ralph-loop iterates rapidly without pause, it can overwhelm external services, exhaust API rate limits, or create excessive noise in commit history. This release adds a `Loop cooldown: N` CLAUDE.md directive that inserts a minimum wait between iterations, ensuring the loop does not start the next iteration until at least N seconds have elapsed since the previous one ended.

## Goals

- Add `Loop cooldown: <seconds>` CLAUDE.md directive
- After each iteration completes, if less than N seconds have elapsed since iteration end, wait before starting the next
- Announce the wait: `[hands-free] Cooldown: waiting <X>s before next iteration`
- Add directive to Available Persistent Settings table
- Bump version to 2.37.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.37.0`
- `grep "\[2.37.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop cooldown in Ralph Loop Integration

**Description:** As a skill user, I want a minimum pause between iterations so I don't hammer external services.

**Acceptance Criteria:**
- [ ] New `### Loop Cooldown` sub-section added under `## Ralph Loop Integration` (after `### Loop Pre-Iteration Hook`)
- [ ] After each iteration ends, hands-free checks elapsed time since iteration end; if less than N seconds, waits the remainder
- [ ] Announce during wait: `[hands-free] Cooldown: waiting <X>s before next iteration`
- [ ] Cooldown does not apply if the loop is paused or stopped — only between active iterations
- [ ] Default: absent (no cooldown, iterations start immediately)
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop cooldown

**Description:** As a skill user, I want to configure the cooldown in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop cooldown: N` row added to Available Persistent Settings table
- [ ] Effect description notes the minimum wait in seconds between iteration end and next iteration start
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.37.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the cooldown feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.36.0` to `2.37.0`
- [ ] New `## [2.37.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: cooldown directive, wait announcement, no cooldown on pause/stop
- [ ] `grep "^version:" SKILL.md` shows `2.37.0`
- [ ] `grep "\[2.37.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop cooldown: N` sets a minimum N-second pause between iteration end and next iteration start
- FR-2: If the previous iteration took longer than N seconds, the next iteration starts immediately (no artificial delay)
- FR-3: Announce the remaining wait time when a cooldown is actually applied
- FR-4: Cooldown does not apply when the loop is paused or manually resumed — only between automatic iterations
- FR-5: Cooldown of 0 or absent means no wait (immediate start)

## Non-Goals

- Per-phase cooldowns within a single iteration
- Dynamic cooldown based on iteration results
- Cooldown between pre-iteration hook and main work

## Technical Considerations

- US-001 adds `### Loop Cooldown` after `### Loop Pre-Iteration Hook` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Cooldown directive documented with announcement format
- No-artificial-delay rule clear (only waits remaining time, not full N seconds if iteration was long)
- Available Persistent Settings table updated
- Even code-fence count, version 2.37.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
