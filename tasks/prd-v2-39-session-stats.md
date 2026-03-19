[PRD]
# PRD: Hands-Free v2.39.0 — Loop Session Stats

## Overview

After a ralph-loop completes (promise met, max-iterations reached, or user stops it), users often want to see an aggregate summary of the session — total iterations, total commits, pass/fail rate, total elapsed time. This release adds a `Loop session stats: on` directive that outputs a session-level aggregate summary when the loop ends.

## Goals

- Add `Loop session stats: on/off` CLAUDE.md directive
- Output an aggregate session summary when the loop ends for any reason
- Include: total iterations, completed/skipped/failed counts, total commits, final test state, elapsed time
- Add directive to Available Persistent Settings table
- Bump version to 2.39.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.39.0`
- `grep "\[2.39.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop session stats in Ralph Loop Integration

**Description:** As a skill user, I want an aggregate session summary at loop end so I can review overall progress.

**Acceptance Criteria:**
- [ ] New `### Loop Session Stats` sub-section added under `## Ralph Loop Integration` (after `### Loop Iteration Summary`)
- [ ] Session stats output format documented in the section
- [ ] Summary fires when the loop ends for any reason (promise met, max-iterations, user stop, HARD STOP)
- [ ] Default: off
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for session stats

**Description:** As a skill user, I want to configure session stats in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop session stats: on/off` row added to Available Persistent Settings table
- [ ] Effect description notes session-level aggregate (distinct from per-iteration summary)
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.39.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the session stats feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.38.0` to `2.39.0`
- [ ] New `## [2.39.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: session stats directive, aggregate at loop end, distinct from iteration summary
- [ ] `grep "^version:" SKILL.md` shows `2.39.0`
- [ ] `grep "\[2.39.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop session stats: on` enables session-level aggregate summary at loop end
- FR-2: Summary fires when the loop terminates for any reason
- FR-3: Summary includes: total iterations, completed/skipped/failed counts, total commits, final test state, total elapsed time
- FR-4: Stats are output to the conversation (not to a file)
- FR-5: Compatible with `Loop iteration summary: on` — both can be enabled simultaneously

## Non-Goals

- Per-iteration summary (use `Loop iteration summary: on`)
- Writing stats to a file (use `Loop notes: on`)
- Real-time stats dashboard during loop execution

## Technical Considerations

- US-001 adds `### Loop Session Stats` after `### Loop Iteration Summary` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- The stats format uses a code block (adds 2 fences)

## Success Metrics

- Session stats documented with output format
- Distinction from Loop iteration summary (end-of-loop vs end-of-each-iteration) clear
- Available Persistent Settings table updated
- Even code-fence count, version 2.39.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
