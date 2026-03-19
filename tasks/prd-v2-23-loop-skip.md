[PRD]
# PRD: Hands-Free v2.23.0 — Loop Skip Command

## Overview

When a loop iteration gets stuck but the auto-stop conditions haven't triggered yet, the user has no way to abandon the current iteration's remaining work and move directly to the next one without cancelling the whole loop. This release adds `/hands-free loop-skip` which discards the current iteration's remaining work and signals to ralph-loop to proceed to the next iteration.

## Goals

- Add `/hands-free loop-skip` command: immediately stops the current iteration and moves to the next
- Document when to use loop-skip vs. loop-pause vs. /hands-free pause
- Add to Quick Reference table and Commands block
- Bump version to 2.23.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.23.0`
- `grep "\[2.23.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Add loop-skip to Quick Reference and Commands block

**Description:** As a skill user, I want to find `/hands-free loop-skip` in the commands reference so I know it's available.

**Acceptance Criteria:**
- [ ] `/hands-free loop-skip` row added to Quick Reference table: "discard remaining work in current iteration, advance to next"
- [ ] `/hands-free loop-skip` line added to Commands block with comment: "skip remaining work in current iteration and advance to next"
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Document loop-skip behavior in Ralph Loop Integration

**Description:** As a skill user, I want the loop-skip behavior explained alongside loop-pause so I can choose the right command.

**Acceptance Criteria:**
- [ ] `loop-skip` row added to the `### Loop Pause and Resume Commands` table:
  - Command: `/hands-free loop-skip`
  - Behavior: immediately stops the current iteration's remaining work; does NOT commit in-progress changes; announces `[hands-free] Iteration skipped — advancing to next iteration` and signals loop to start the next iteration
- [ ] Note added: skipped iterations do NOT consume the completion promise check — the promise is re-evaluated at the start of the next iteration
- [ ] Note added: if `/hands-free loop-skip` is invoked outside loop-aware mode, announce: `[hands-free] loop-skip has no effect outside loop mode`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.23.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the loop-skip feature.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.22.0` to `2.23.0`
- [ ] New `## [2.23.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: loop-skip command, behavior (discard iteration, advance to next, no commit), comparison with loop-pause
- [ ] `grep "^version:" SKILL.md` shows `2.23.0`
- [ ] `grep "\[2.23.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `/hands-free loop-skip` stops all pending work in the current iteration immediately
- FR-2: In-progress file changes (unstaged) are NOT committed when skipping — the user is responsible for those; announce a warning if unstaged changes exist
- FR-3: The loop advances to the next iteration immediately after skip
- FR-4: Skipped iterations are logged: `[loop-skip] iteration N skipped — advancing`
- FR-5: The completion promise is re-evaluated fresh at the start of the next iteration

## Non-Goals

- Stashing uncommitted changes on skip (the user manages local state)
- Auto-skip based on time limits
- Skipping multiple iterations at once

## Technical Considerations

- US-001 modifies the Quick Reference table and Commands code block
- US-002 adds a row to the `### Loop Pause and Resume Commands` table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- loop-skip documented in quick reference, commands block, and loop integration section
- Behavior distinction from loop-pause clearly explained
- Even code-fence count, version 2.23.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
