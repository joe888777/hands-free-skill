[PRD]
# PRD: Hands-Free v2.30.0 — Loop Auto-Push

## Overview

When running a long ralph-loop with auto-commit enabled, changes accumulate locally but are never pushed to the remote until the user reaches finishing-a-development-branch. This release adds a `Loop auto-push: on` directive that causes hands-free to automatically push to origin after each iteration's auto-commits complete, keeping the remote in sync throughout the loop.

## Goals

- Add `Loop auto-push: on/off` CLAUDE.md directive
- When enabled (with auto-commit also on), push to the remote at the end of each successful iteration
- Skip the push on loop-skip, HARD STOP, or when no new commits were made
- Document the skip conditions and the dependency on auto-commit
- Add directive to Available Persistent Settings table
- Bump version to 2.30.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.30.0`
- `grep "\[2.30.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop auto-push behavior in Ralph Loop Integration

**Description:** As a skill user, I want the remote to stay in sync with local iteration commits without manual pushes.

**Acceptance Criteria:**
- [ ] New `### Loop Auto-Push` sub-section added under `## Ralph Loop Integration` (after `### Loop Iteration Notes`)
- [ ] Section describes: when `Loop auto-push: on` and `Auto-commit: on`, run `git push` after each iteration's commits
- [ ] Announce: `[hands-free] Auto-push: pushed N commits to origin/<branch>`
- [ ] Skip conditions documented: loop-skip, HARD STOP, no new commits, push failure (non-halting error)
- [ ] Dependency on auto-commit noted: if auto-commit is off, auto-push is silently disabled
- [ ] Default: off
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop auto-push

**Description:** As a skill user, I want to configure loop auto-push in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop auto-push: on/off` row added to Available Persistent Settings table
- [ ] Effect description includes dependency on Auto-commit: on
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.30.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the auto-push feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.29.0` to `2.30.0`
- [ ] New `## [2.30.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: auto-push directive, skip conditions, auto-commit dependency
- [ ] `grep "^version:" SKILL.md` shows `2.30.0`
- [ ] `grep "\[2.30.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop auto-push: on` triggers `git push` at the end of each iteration after auto-commits complete
- FR-2: Push is skipped when the iteration was skipped via loop-skip
- FR-3: Push is skipped when the iteration ended with a HARD STOP
- FR-4: Push is skipped when no new commits were made in the iteration
- FR-5: If `git push` fails, announce the error and continue to the next iteration without halting the loop
- FR-6: `Auto-commit: off` silently disables auto-push regardless of the auto-push setting

## Non-Goals

- Force-pushing or rebase-and-push (only plain `git push`)
- Per-file push scoping
- Push to non-origin remotes

## Technical Considerations

- US-001 adds `### Loop Auto-Push` after `### Loop Iteration Notes` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Auto-push directive documented with skip conditions
- Auto-commit dependency explicit
- Push failure non-halting behavior clear
- Available Persistent Settings table updated
- Even code-fence count, version 2.30.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
