[PRD]
# PRD: Hands-Free v2.47.0 — Loop Iteration Timeout

## Overview

Individual loop iterations can occasionally hang or run much longer than expected — for example, when a test suite waits for a network timeout, a build process stalls, or systematic-debugging enters an unusually long cycle. This release adds a `Loop iteration timeout: N` directive that fires a HARD STOP if a single iteration has been running for more than N minutes.

## Goals

- Add `Loop iteration timeout: N` CLAUDE.md directive
- When set, track elapsed time since iteration start
- If elapsed time exceeds N minutes, fire a HARD STOP
- Add directive to Available Persistent Settings table
- Bump version to 2.47.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.47.0`
- `grep "\[2.47.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document Loop iteration timeout in Ralph Loop Integration

**Description:** As a skill user, I want the loop to stop if a single iteration runs too long so I can detect hangs and stalls before they consume all remaining iterations.

**Acceptance Criteria:**
- [ ] New `### Loop Iteration Timeout` sub-section added under `## Ralph Loop Integration` (after `### Loop Max Changes`, before `### What Hands-Free Does NOT Do in Loop Mode`)
- [ ] Documents that elapsed time is measured from the start of the iteration
- [ ] HARD STOP message documented: `[hands-free] HARD STOP — Iteration timeout: iteration has been running for N minutes (limit: M minutes). Run /hands-free resume to continue to the next iteration.`
- [ ] Documents that after `/hands-free resume`, the current iteration is abandoned and the loop moves to the next iteration
- [ ] Documents that the check is opportunistic — checked at natural pause points (before and after tool calls, before auto-commit), not via a background timer
- [ ] If absent, no per-iteration time limit is enforced
- [ ] Default: absent — no limit
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop iteration timeout

**Description:** As a skill user, I want to configure the iteration timeout in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop iteration timeout: N` row added to Available Persistent Settings table
- [ ] Effect description notes the HARD STOP trigger and that resume moves to the next iteration
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.47.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the iteration timeout feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.46.0` to `2.47.0`
- [ ] New `## [2.47.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: iteration timeout directive, HARD STOP trigger, opportunistic check, resume behavior
- [ ] `grep "^version:" SKILL.md` shows `2.47.0`
- [ ] `grep "\[2.47.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop iteration timeout: N` sets the maximum duration (in minutes) for a single iteration
- FR-2: Elapsed time is measured from the start of the iteration
- FR-3: The check is opportunistic — evaluated at natural pause points (before/after major tool calls, before auto-commit), not via a background timer
- FR-4: If elapsed time > N minutes, fire HARD STOP: `[hands-free] HARD STOP — Iteration timeout: iteration has been running for N minutes (limit: M minutes). Run /hands-free resume to continue to the next iteration.`
- FR-5: After `/hands-free resume`, the current iteration is abandoned; the loop proceeds to the next iteration (the timed-out iteration is not retried)
- FR-6: If absent, no per-iteration time check is performed

## Non-Goals

- Background timer running independently of Claude's execution
- Killing running subprocesses (only the iteration flow is halted)
- Per-phase timeouts (single limit applies to the whole iteration)

## Technical Considerations

- US-001 adds `### Loop Iteration Timeout` after `### Loop Max Changes` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- No new code fences needed

## Success Metrics

- Opportunistic check timing documented
- Resume behavior (move to next iteration) documented
- Available Persistent Settings table updated
- Even code-fence count, version 2.47.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
