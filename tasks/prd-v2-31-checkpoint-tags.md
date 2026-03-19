[PRD]
# PRD: Hands-Free v2.31.0 — Loop Checkpoint Tags

## Overview

When a long ralph-loop modifies many files over many iterations, users sometimes want to revert to the state at the end of a specific iteration. This release adds a `Loop checkpoint tags: on` directive that causes hands-free to create a lightweight git tag `loop-iter-N` after each iteration's auto-commits complete, giving the user easy restore points.

## Goals

- Add `Loop checkpoint tags: on/off` CLAUDE.md directive
- When enabled, create tag `loop-iter-N` after each iteration's commits (before auto-push)
- Document the tag format, restore workflow, and the fact that tags are not auto-pushed
- Add directive to Available Persistent Settings table
- Bump version to 2.31.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.31.0`
- `grep "\[2.31.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop checkpoint tags in Ralph Loop Integration

**Description:** As a skill user, I want hands-free to tag each iteration's state so I can easily revert if a later iteration goes wrong.

**Acceptance Criteria:**
- [ ] New `### Loop Checkpoint Tags` sub-section added under `## Ralph Loop Integration` (after `### Loop Auto-Push`)
- [ ] Tag format: `loop-iter-<N>` where N is the iteration number
- [ ] Tags created after auto-commits, before auto-push
- [ ] Tags are NOT auto-pushed; user must `git push origin --tags` manually if needed
- [ ] Announce: `[hands-free] Checkpoint tag: loop-iter-N`
- [ ] If no new commits were made in the iteration, no tag is created
- [ ] Default: off
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for checkpoint tags

**Description:** As a skill user, I want to configure loop checkpoint tags in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop checkpoint tags: on/off` row added to Available Persistent Settings table
- [ ] Effect description notes: tags not auto-pushed
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.31.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the checkpoint tags feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.30.0` to `2.31.0`
- [ ] New `## [2.31.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: checkpoint tags directive, format, not auto-pushed
- [ ] `grep "^version:" SKILL.md` shows `2.31.0`
- [ ] `grep "\[2.31.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop checkpoint tags: on` creates a lightweight git tag after each iteration's auto-commits
- FR-2: Tag format: `loop-iter-N` (e.g., `loop-iter-1`, `loop-iter-15`)
- FR-3: Tag is created AFTER auto-commits but BEFORE auto-push
- FR-4: Tags are NOT pushed automatically (user responsibility)
- FR-5: If the iteration produced no new commits, no tag is created for that iteration
- FR-6: Announce `[hands-free] Checkpoint tag: loop-iter-N` when a tag is created

## Non-Goals

- Annotated tags (lightweight only)
- Auto-pushing checkpoint tags
- Pruning old checkpoint tags automatically

## Technical Considerations

- US-001 adds `### Loop Checkpoint Tags` after `### Loop Auto-Push` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- Checkpoint tags documented with format and lifecycle
- Not-auto-pushed behavior clear
- No-new-commits skip condition documented
- Available Persistent Settings table updated
- Even code-fence count, version 2.31.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
