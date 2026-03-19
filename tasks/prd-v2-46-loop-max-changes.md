[PRD]
# PRD: Hands-Free v2.46.0 — Loop Max Changes

## Overview

In a well-functioning loop, each iteration makes a small, focused set of changes. If an iteration modifies an unexpectedly large number of files, it may indicate that the loop is taking a wrong approach — rewriting unrelated files, applying a broad refactor, or running amok. This release adds a `Loop max changes: N` directive that fires a HARD STOP if the number of files changed in a single iteration exceeds N.

## Goals

- Add `Loop max changes: N` CLAUDE.md directive
- When set, count changed files at the end of each iteration (after all edits but before auto-commit)
- If the count exceeds N, fire a HARD STOP
- Add directive to Available Persistent Settings table
- Bump version to 2.46.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.46.0`
- `grep "\[2.46.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document Loop max changes in Ralph Loop Integration

**Description:** As a skill user, I want the loop to stop if it changes too many files so I can review whether the scope is correct before continuing.

**Acceptance Criteria:**
- [ ] New `### Loop Max Changes` sub-section added under `## Ralph Loop Integration` (after `### Loop Commit Prefix`, before `### What Hands-Free Does NOT Do in Loop Mode`)
- [ ] Documents that "changed files" = files reported by `git diff --name-only HEAD` (staged + unstaged changes against HEAD)
- [ ] HARD STOP message documented: `[hands-free] HARD STOP — Max changes exceeded: N files changed in this iteration (limit: M). Run /hands-free resume to commit and continue anyway.`
- [ ] Documents that the check runs after all edits complete, before auto-commit
- [ ] Documents that new untracked files are included in the count (via `git status --short`)
- [ ] If absent, no file-count limit is enforced
- [ ] Default: absent — no limit
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop max changes

**Description:** As a skill user, I want to configure the max changes limit in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop max changes: N` row added to Available Persistent Settings table
- [ ] Effect description notes the HARD STOP trigger and that new untracked files are counted
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.46.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the max changes feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.45.0` to `2.46.0`
- [ ] New `## [2.46.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: max changes directive, HARD STOP trigger, file count method
- [ ] `grep "^version:" SKILL.md` shows `2.46.0`
- [ ] `grep "\[2.46.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop max changes: N` sets the maximum number of files that may be changed in a single iteration
- FR-2: Count = number of lines in `git status --short` output (includes staged, unstaged, and new untracked files; excludes `.gitignore`d files)
- FR-3: Check runs after all edits complete, before auto-commit
- FR-4: If count > N, fire HARD STOP: `[hands-free] HARD STOP — Max changes exceeded: N files changed in this iteration (limit: M). Run /hands-free resume to commit and continue anyway.`
- FR-5: After `/hands-free resume`, the iteration proceeds to auto-commit despite the excess; the limit is not re-checked for the same iteration
- FR-6: If absent, no file-count check is performed

## Non-Goals

- Counting lines changed (only file count)
- Per-directory limits
- Warning thresholds below the hard limit (only one threshold: the limit itself)

## Technical Considerations

- US-001 adds `### Loop Max Changes` after `### Loop Commit Prefix` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- No new code fences needed

## Success Metrics

- File count method (git status --short) documented
- Timing of check (after edits, before auto-commit) documented
- Resume behavior documented
- Available Persistent Settings table updated
- Even code-fence count, version 2.46.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
