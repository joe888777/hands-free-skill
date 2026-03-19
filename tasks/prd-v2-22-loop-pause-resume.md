[PRD]
# PRD: Hands-Free v2.22.0 — Loop Pause/Resume Commands

## Overview

The `/hands-free pause` and `/hands-free resume` commands exist for skill-level approval points but are not documented in the context of loop mode. When a loop is running, users may want to temporarily pause iteration work (e.g., to manually fix a stubborn bug mid-loop) and then resume, without cancelling the loop entirely. This release adds loop-specific pause/resume documentation and a `/hands-free loop-pause` variant that pauses at the next natural iteration boundary rather than immediately.

## Goals

- Document `/hands-free loop-pause` command: pauses the loop after the current iteration completes (not mid-iteration)
- Document `/hands-free loop-resume` command: resumes the loop from the next iteration
- Distinguish between immediate pause (existing `/hands-free pause`) and boundary pause (new `loop-pause`)
- Add both commands to the `## Commands` quick-reference table
- Bump version to 2.22.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.22.0`
- `grep "\[2.22.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Add loop-pause and loop-resume to the Commands table

**Description:** As a skill user, I want to see `/hands-free loop-pause` and `/hands-free loop-resume` listed in the commands reference so I know these options exist.

**Acceptance Criteria:**
- [ ] `/hands-free loop-pause` row added to the `## Commands` block: "pause after current iteration completes (boundary pause)"
- [ ] `/hands-free loop-resume` row added to the `## Commands` block: "resume the loop from next iteration after loop-pause"
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Specify loop-pause behavior in Ralph Loop Integration

**Description:** As a skill user, I want the loop integration section to explain loop-pause vs. immediate-pause behavior so I can choose the right command.

**Acceptance Criteria:**
- [ ] New paragraph or note added under `### Loop-Aware Behavior` (or at end of the section) distinguishing:
  - `/hands-free pause` — immediate; stops auto-accept at the next approval point mid-iteration
  - `/hands-free loop-pause` — boundary; completes the current iteration's work, then pauses before the next iteration begins; announces `[hands-free] Loop-pause armed — will pause after this iteration completes`
  - `/hands-free loop-resume` — resumes from the next iteration; announces `[hands-free] Loop-pause cleared — resuming loop`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.22.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the loop pause/resume commands.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.21.0` to `2.22.0`
- [ ] New `## [2.22.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: loop-pause (boundary pause after current iteration), loop-resume, distinction from immediate pause
- [ ] `grep "^version:" SKILL.md` shows `2.22.0`
- [ ] `grep "\[2.22.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `/hands-free loop-pause` arms a flag; the current iteration completes normally; before the next iteration begins, hands-free pauses and announces `[hands-free] Loop-paused — type /hands-free loop-resume to continue`
- FR-2: `/hands-free loop-resume` clears the flag; the next iteration proceeds
- FR-3: If `/hands-free loop-pause` is called when not in loop-aware mode, announce: `[hands-free] loop-pause has no effect outside loop mode`
- FR-4: The loop-pause state is reflected in `/hands-free status` as `Loop-pause: armed` or `Loop-pause: off`

## Non-Goals

- Cancelling the ralph-loop entirely (that is ralph-loop's responsibility)
- Persisting loop-pause state across sessions
- Auto-arming loop-pause based on conditions

## Technical Considerations

- US-001 modifies the `## Commands` code block
- US-002 adds text to the `### Loop-Aware Behavior` sub-section
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- loop-pause and loop-resume commands documented in commands table
- Behavior distinction from immediate-pause clearly explained
- Even code-fence count, version 2.22.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
