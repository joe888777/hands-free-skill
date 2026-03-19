[PRD]
# PRD: Hands-Free v2.15.0 — Loop Auto-Stop Conditions

## Overview

The loop currently runs until `max_iterations` is reached or the completion promise is met, with no mechanism to pause when the workspace enters a consistently unhealthy or stagnant state. This release adds two configurable auto-stop conditions — a health floor breach and a zero-progress ceiling — that halt the current iteration and notify the user for review without consuming the completion promise or terminating the loop itself.

## Goals

- Detect and halt on health floor breach: `health_score < 25` for 3 consecutive iterations
- Detect and halt on zero-progress ceiling: `velocity_trend` contains 5 consecutive zeros
- Allow both conditions to be disabled or reconfigured via CLAUDE.md
- Log auto-stop events in the session log with the triggering condition
- Bump version to 2.15.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.15.0`
- `grep "\[2.15.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Specify auto-stop conditions in Loop-Aware Behavior

**Description:** As a skill user, I want hands-free to automatically halt an iteration when the loop is stuck or unhealthy so that I am notified before wasted work accumulates.

**Acceptance Criteria:**
- [ ] New `### Loop Auto-Stop Conditions` sub-section added under `## Ralph Loop Integration`
- [ ] Health Floor condition specified: if `health_score < 25` for 3 consecutive iterations → announce `[hands-free] LOOP AUTO-STOP: Health floor breached (score: N for 3 iterations). Pausing for user review.` and halt the current iteration
- [ ] Zero-Progress condition specified: if all 5 entries of `velocity_trend` are `0` → announce `[hands-free] LOOP AUTO-STOP: No story progress in last 5 iterations. Pausing for user review.` and halt the current iteration
- [ ] Halting the iteration means: stop all work for this iteration, do not output the completion promise, log the auto-stop reason in the session log
- [ ] ralph-loop itself is NOT terminated — it continues to the next iteration after the user resumes
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Specify CLAUDE.md overrides for auto-stop conditions

**Description:** As a skill user, I want to disable or reconfigure auto-stop conditions via CLAUDE.md so that I can tune thresholds for my specific project.

**Acceptance Criteria:**
- [ ] `- Loop auto-stop: off` in CLAUDE.md disables both auto-stop conditions entirely
- [ ] `- Loop health floor: 15` in CLAUDE.md overrides the default health floor threshold from 25 to the specified value
- [ ] Override reference added to the `## CLAUDE.md Override Reference` section's "Available Persistent Settings" table
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.15.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the loop auto-stop feature.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.14.0` to `2.15.0`
- [ ] New `## [2.15.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: health floor auto-stop (3 consecutive iterations below threshold), zero-progress auto-stop (5 consecutive zero-velocity iterations), CLAUDE.md overrides
- [ ] `grep "^version:" SKILL.md` shows `2.15.0`
- [ ] `grep "\[2.15.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: Health floor check runs at the start of each iteration using `health_score` from the checkpoint; requires 3 consecutive readings below the floor (tracked in session memory)
- FR-2: Zero-progress check runs at the start of each iteration using `velocity_trend`; triggers when all 5 entries are `0`
- FR-3: Auto-stop announces a specific message per condition and adds an entry to the session log: `[auto-stop] <condition> — iteration N halted`
- FR-4: After auto-stop, resuming is done by the user typing "continue"
- FR-5: `Loop auto-stop: off` disables both conditions; `Loop health floor: N` only overrides the health floor threshold
- FR-6: Default health floor threshold: 25; default consecutive count: 3

## Non-Goals

- Terminating ralph-loop entirely based on auto-stop conditions
- Persisting consecutive-low-health counter to the checkpoint
- Configuring the zero-progress window via CLAUDE.md
- Auto-fixing the underlying cause of the auto-stop

## Technical Considerations

- US-001 adds `### Loop Auto-Stop Conditions` sub-section under `## Ralph Loop Integration` (after `### Loop Health Score`)
- US-002 adds rows to the "Available Persistent Settings" table in `## CLAUDE.md Override Reference`
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Two auto-stop conditions specified with exact trigger criteria and announcement formats
- CLAUDE.md override reference updated
- Even code-fence count, version 2.15.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
