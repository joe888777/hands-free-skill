[PRD]
# PRD: Hands-Free v2.29.0 — Loop Iteration Notes

## Overview

In long ralph-loop runs the user often wants a persistent per-iteration record to review what happened after the loop finishes. This release adds a `Loop notes: on` CLAUDE.md directive that causes hands-free to append a brief structured summary for each iteration to `.claude/loop-notes.md`, capturing status, changes, test outcome, warnings, and duration.

## Goals

- Add `Loop notes: on/off` CLAUDE.md directive
- When enabled, append a structured entry to `.claude/loop-notes.md` after each iteration completes
- Document the entry format and the fact that the file is a session artifact (not auto-committed)
- Add directive to Available Persistent Settings table
- Bump version to 2.29.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.29.0`
- `grep "\[2.29.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop iteration notes in Ralph Loop Integration

**Description:** As a skill user, I want hands-free to write a per-iteration summary so I can review what happened across a long loop run.

**Acceptance Criteria:**
- [ ] New `### Loop Iteration Notes` sub-section added under `## Ralph Loop Integration` (after `### Loop Max Consecutive Failures`)
- [ ] Section describes: when `Loop notes: on`, append a structured entry to `.claude/loop-notes.md` after each iteration
- [ ] Entry format documented (iteration number, timestamp, status, files changed, test outcome, warnings, duration)
- [ ] Notes file is NOT auto-committed — it is a session artifact; note recommends adding to `.gitignore`
- [ ] Default: off
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop notes

**Description:** As a skill user, I want to configure loop iteration notes in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop notes: on/off` row added to Available Persistent Settings table
- [ ] Effect: appends per-iteration summary to `.claude/loop-notes.md`; default: `off`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.29.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the iteration notes feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.28.0` to `2.29.0`
- [ ] New `## [2.29.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: notes directive, entry format, file not auto-committed
- [ ] `grep "^version:" SKILL.md` shows `2.29.0`
- [ ] `grep "\[2.29.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop notes: on` enables per-iteration summary writing after each iteration
- FR-2: Each entry is appended (not overwritten) to `.claude/loop-notes.md`
- FR-3: Entry captures: iteration number, timestamp, completion status, number of files changed and commits, test outcome, any warnings that fired, approximate duration
- FR-4: The notes file is excluded from auto-commit — it is a session artifact
- FR-5: `Loop notes: off` (default) — no file is written

## Non-Goals

- Structured machine-parseable log format (the notes are human-readable)
- Committing loop-notes.md automatically
- Per-event logging (use `/hands-free log` for that)

## Technical Considerations

- US-001 adds `### Loop Iteration Notes` after `### Loop Max Consecutive Failures` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Iteration notes directive documented with entry format
- Session-artifact nature clear (not auto-committed)
- Available Persistent Settings table updated
- Even code-fence count, version 2.29.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
