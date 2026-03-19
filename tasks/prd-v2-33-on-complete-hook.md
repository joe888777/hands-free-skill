[PRD]
# PRD: Hands-Free v2.33.0 — Loop On-Complete Hook

## Overview

When a long ralph-loop finishes, users often want to trigger a notification, run cleanup, or kick off a follow-up process automatically. This release adds a `Loop on-complete: <command>` CLAUDE.md directive that runs a specified shell command once immediately before the completion promise is output.

## Goals

- Add `Loop on-complete: <command>` CLAUDE.md directive
- When the promise condition is met, run the command once before outputting the promise
- Apply the same cwd-scope and HARD STOP rules as all other shell commands
- If the command fails, announce the error but still output the completion promise
- Add directive to Available Persistent Settings table
- Bump version to 2.33.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.33.0`
- `grep "\[2.33.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop on-complete hook in Ralph Loop Integration

**Description:** As a skill user, I want a hook that runs when my loop finishes so I can send notifications or trigger follow-up processes.

**Acceptance Criteria:**
- [ ] New `### Loop On-Complete Hook` sub-section added under `## Ralph Loop Integration` (after `### Loop Max Duration`)
- [ ] Hook runs once immediately before the completion promise is output
- [ ] Announce: `[hands-free] On-complete hook: running '<command>'`
- [ ] Same cwd-scope and HARD STOP rules apply as all other shell commands
- [ ] If the command fails (non-zero exit), announce the error and still output the promise
- [ ] Hook runs exactly once per loop session on first promise satisfaction
- [ ] Default: absent
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for on-complete hook

**Description:** As a skill user, I want to configure the on-complete hook in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop on-complete: <command>` row added to Available Persistent Settings table
- [ ] Effect description notes that the same shell command rules apply; promise still output on failure
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.33.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the on-complete hook feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.32.0` to `2.33.0`
- [ ] New `## [2.33.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: on-complete hook, execution rules, failure-tolerant
- [ ] `grep "^version:" SKILL.md` shows `2.33.0`
- [ ] `grep "\[2.33.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop on-complete: <command>` registers a one-shot hook for loop completion
- FR-2: Hook fires immediately before the completion promise is output
- FR-3: Same shell auto-pass/HARD STOP rules apply as any other shell command
- FR-4: Hook failure (non-zero exit) is announced but does not suppress the promise output
- FR-5: Hook fires exactly once per loop session (not on every iteration)

## Non-Goals

- Multiple on-complete hooks
- On-failure hooks (separate feature)
- Passing loop state as arguments to the hook command

## Technical Considerations

- US-001 adds `### Loop On-Complete Hook` after `### Loop Max Duration` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md

## Success Metrics

- On-complete hook documented with timing and failure behavior
- Security rules (cwd-scope, HARD STOP) explicitly noted
- One-shot per session guarantee clear
- Available Persistent Settings table updated
- Even code-fence count, version 2.33.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
