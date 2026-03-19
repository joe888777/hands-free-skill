[PRD]
# PRD: Hands-Free v2.18.0 — Loop-Mode Troubleshooting Entries

## Overview

The `## Troubleshooting` section covers general hands-free issues but has no entries specific to loop mode. This release adds a dedicated group of loop-mode Q&A entries covering the most common issues users encounter: stuck iterations, checkpoint corruption, auto-stop false positives, velocity stall false positives, and health score anomalies.

## Goals

- Add 5 loop-mode troubleshooting entries to `## Troubleshooting`
- Bump version to 2.18.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.18.0`
- `grep "\[2.18.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Add loop-mode troubleshooting entries to Troubleshooting section

**Description:** As a skill user, I want Q&A entries covering common loop-mode failure modes so that I can self-diagnose issues without reading the full skill.

**Acceptance Criteria:**
- [ ] New group heading `### Loop Mode Issues` added at the end of `## Troubleshooting` section
- [ ] Entry: "The loop keeps re-brainstorming instead of resuming where it left off" — checkpoint SHA mismatch or missing checkpoint; how to recover
- [ ] Entry: "Auto-stop fired but my health score looks fine" — consecutive count tracked in session memory only; restarting session resets counter; how to disable with `Loop auto-stop: off`
- [ ] Entry: "Velocity stall warning fires even though stories are completing" — `stories_completed_this_iteration` may be 0 if `completed_stories` list wasn't updated in the checkpoint; how to inspect the checkpoint
- [ ] Entry: "Health score is always 50 even with passing tests" — `test_summary` field missing from checkpoint; indicates no test runner output was captured; how to verify
- [ ] Entry: "The loop re-brainstorms every iteration even with a fresh checkpoint" — `pending_stories` field empty or `active_plan_file` not set; checkpoint state not being written correctly; check pre-flight for BLOCKED state
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Bump version to 2.18.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the loop troubleshooting additions.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.17.0` to `2.18.0`
- [ ] New `## [2.18.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: 5 loop-mode troubleshooting entries added under new "Loop Mode Issues" group
- [ ] `grep "^version:" SKILL.md` shows `2.18.0`
- [ ] `grep "\[2.18.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: All 5 troubleshooting entries include: symptom heading, root cause explanation, and recovery steps
- FR-2: Entries are grouped under `### Loop Mode Issues` heading at end of `## Troubleshooting`
- FR-3: Recovery steps reference specific checkpoint fields, CLAUDE.md overrides, or hands-free commands where applicable

## Non-Goals

- Adding troubleshooting entries for non-loop issues
- Adding automated diagnostics commands

## Technical Considerations

- US-001 appends to `## Troubleshooting` section; find end of Troubleshooting and insert before the next `##` heading
- US-002 modifies frontmatter and CHANGELOG.md
- No code fences are added in the troubleshooting text (maintains even fence count)

## Success Metrics

- 5 loop-mode Q&A entries added under grouped heading
- Even code-fence count, version 2.18.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
