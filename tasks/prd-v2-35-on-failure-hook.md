[PRD]
# PRD: Hands-Free v2.35.0 — Loop On-Failure Hook

## Overview

When an iteration ends with test failures, users often want to trigger a notification or run a diagnostic command automatically. This release adds a `Loop on-failure: <command>` CLAUDE.md directive that runs a specified shell command after each iteration that ends with one or more failing tests.

## Goals

- Add `Loop on-failure: <command>` CLAUDE.md directive
- Run the command after each iteration that ends with test failures
- Apply the same cwd-scope and HARD STOP rules as all other shell commands
- If the command fails, announce the error but continue loop execution
- Add directive to Available Persistent Settings table
- Bump version to 2.35.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.35.0`
- `grep "\[2.35.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop on-failure hook in Ralph Loop Integration

**Description:** As a skill user, I want a hook that runs when an iteration fails so I can send notifications or trigger diagnostics.

**Acceptance Criteria:**
- [ ] New `### Loop On-Failure Hook` sub-section added under `## Ralph Loop Integration` (after `### Loop Skip-if-Unchanged`)
- [ ] Hook runs after each iteration that ends with one or more failing tests
- [ ] Announce: `[hands-free] On-failure hook: running '<command>'`
- [ ] Same cwd-scope and HARD STOP rules apply as all other shell commands
- [ ] If the command fails (non-zero exit), announce the error and continue loop execution
- [ ] Hook fires once per failed iteration (may fire multiple times across iterations, unlike on-complete)
- [ ] Default: absent
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for on-failure hook

**Description:** As a skill user, I want to configure the on-failure hook in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop on-failure: <command>` row added to Available Persistent Settings table
- [ ] Effect description notes that it fires per failed iteration (not just once), same shell rules apply, non-blocking on hook failure
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.35.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the on-failure hook feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.34.0` to `2.35.0`
- [ ] New `## [2.35.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: on-failure hook, per-iteration firing, non-blocking
- [ ] `grep "^version:" SKILL.md` shows `2.35.0`
- [ ] `grep "\[2.35.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop on-failure: <command>` registers a per-iteration hook for test failures
- FR-2: Hook fires after each iteration that ends with one or more failing tests
- FR-3: Same shell auto-pass/HARD STOP rules apply as any other shell command
- FR-4: Hook failure (non-zero exit) is announced but does not pause or stop the loop
- FR-5: Hook fires once per failed iteration (can fire on multiple iterations, unlike on-complete)

## Non-Goals

- On-complete hook (already implemented in v2.33.0)
- Passing failure details as arguments to the hook command
- Multiple on-failure hooks

## Technical Considerations

- US-001 adds `### Loop On-Failure Hook` after `### Loop Skip-if-Unchanged` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- On-failure hook documented with timing and failure behavior
- Security rules (cwd-scope, HARD STOP) explicitly noted
- Per-iteration (not one-shot) guarantee clear
- Available Persistent Settings table updated
- Even code-fence count, version 2.35.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
