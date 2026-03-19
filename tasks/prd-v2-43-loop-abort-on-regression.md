[PRD]
# PRD: Hands-Free v2.43.0 — Loop Abort on Regression

## Overview

When a ralph-loop is making progress, the failure count should generally decrease over time. If a later iteration has more failing tests than the previous one, it suggests the latest changes introduced a regression. This release adds a `Loop abort on regression: on` directive that fires a HARD STOP whenever the failing test count increases from one iteration to the next, preventing the loop from compounding regressions.

## Goals

- Add `Loop abort on regression: on/off` CLAUDE.md directive
- When enabled, compare failing test count at the end of each iteration to the previous iteration
- If failing tests increased, fire a HARD STOP before the next iteration starts
- Reset the baseline count after each HARD STOP is resolved and the loop resumes
- Add directive to Available Persistent Settings table
- Bump version to 2.43.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.43.0`
- `grep "\[2.43.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop abort-on-regression in Ralph Loop Integration

**Description:** As a skill user, I want the loop to stop if my changes introduce new failures so I can investigate before the loop digs deeper.

**Acceptance Criteria:**
- [ ] New `### Loop Abort on Regression` sub-section added under `## Ralph Loop Integration` (after `### Loop Quiet Mode`, before `### What Hands-Free Does NOT Do in Loop Mode`)
- [ ] Documents the comparison logic: failing count at end of iteration N vs iteration N-1
- [ ] HARD STOP message documented: `[hands-free] HARD STOP — Regression detected: N failing tests (was M in previous iteration). Run /hands-free resume to continue anyway.`
- [ ] First iteration always passes (no prior baseline to compare against)
- [ ] Baseline resets after user resumes with `/hands-free resume`
- [ ] Compatible with `Loop max failures` — both can be active simultaneously
- [ ] Default: off
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop abort-on-regression

**Description:** As a skill user, I want to configure regression detection in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop abort on regression: on/off` row added to Available Persistent Settings table
- [ ] Effect description notes the comparison logic and HARD STOP trigger
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.43.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the regression detection feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.42.0` to `2.43.0`
- [ ] New `## [2.43.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: abort-on-regression, comparison logic, HARD STOP trigger
- [ ] `grep "^version:" SKILL.md` shows `2.43.0`
- [ ] `grep "\[2.43.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop abort on regression: on` enables regression detection between consecutive iterations
- FR-2: At the end of each iteration, record the failing test count
- FR-3: Before the NEXT iteration starts, compare the recorded count to the previous iteration's count
- FR-4: If current > previous, fire HARD STOP: `[hands-free] HARD STOP — Regression detected: N failing tests (was M in previous iteration). Run /hands-free resume to continue anyway.`
- FR-5: First iteration has no prior baseline — skip the comparison and proceed normally
- FR-6: After user resumes with `/hands-free resume`, reset the baseline to the current failing count so the next iteration compares against the resumed state
- FR-7: Compatible with `Loop max failures` — regression abort and max-failures can both be active

## Non-Goals

- Comparing individual test names (only counts are compared)
- Configurable regression threshold (any increase triggers; threshold is always +1)
- Regression detection mid-iteration (only checked at iteration boundaries)

## Technical Considerations

- US-001 adds `### Loop Abort on Regression` after `### Loop Quiet Mode` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- No new code fences needed

## Success Metrics

- Comparison logic clearly documented
- HARD STOP message documented with resume path
- First-iteration edge case addressed
- Available Persistent Settings table updated
- Even code-fence count, version 2.43.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
