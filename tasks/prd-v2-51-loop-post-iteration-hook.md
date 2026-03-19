[PRD]
# PRD: Hands-Free v2.51.0 — Loop Post-Iteration Hook

## Overview

Symmetric counterpart to the pre-iteration hook added in v2.50.0. Some projects need to run a cleanup or notification command after each iteration completes — for example, updating a status file, notifying a webhook, or running a post-commit lint check. This release adds a `Loop post-iteration hook: <cmd>` directive that runs after auto-commit (and after auto-push if enabled) at the end of each iteration. If the command exits non-zero, a HARD STOP is fired.

## Goals

- Add `Loop post-iteration hook: <cmd>` CLAUDE.md directive
- When set, run the command at the end of each iteration after auto-commit and auto-push
- If exit code is non-zero, fire a HARD STOP
- Add directive to Available Persistent Settings table
- Bump version to 2.51.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.51.0`
- `grep "\[2.51.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document Loop post-iteration hook in Ralph Loop Integration

**Description:** As a skill user, I want to run a cleanup or notification command after each iteration completes so I can update external state or verify post-commit invariants.

**Acceptance Criteria:**
- [ ] New `### Loop Post-Iteration Hook` sub-section added under `## Ralph Loop Integration` (after `### Loop Pre-Iteration Hook`, before `### What Hands-Free Does NOT Do in Loop Mode`)
- [ ] Documents that the command runs after auto-commit and after auto-push (if enabled), at the very end of the iteration
- [ ] Documents that exit code 0 = proceed to next iteration; non-zero = HARD STOP
- [ ] HARD STOP message documented: `[hands-free] HARD STOP — Post-iteration hook failed (exit <N>): <cmd>. Run /hands-free resume to continue to the next iteration.`
- [ ] Documents that after `/hands-free resume`, the loop continues to the next iteration (the hook is not re-run)
- [ ] Documents that if auto-commit is off or no changes were committed, the hook still runs (timing is "end of iteration", not "after commit")
- [ ] Documents that the command is run with cwd-scope rules
- [ ] If absent, no post-iteration hook is run
- [ ] Default: absent
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop post-iteration hook

**Description:** As a skill user, I want to configure the post-iteration hook in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop post-iteration hook: <cmd>` row added to Available Persistent Settings table
- [ ] Effect description notes timing (after auto-commit/auto-push), exit code semantics, and resume behavior
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.51.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the post-iteration hook feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.50.0` to `2.51.0`
- [ ] New `## [2.51.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: post-iteration hook directive, timing, exit code semantics, resume behavior
- [ ] `grep "^version:" SKILL.md` shows `2.51.0`
- [ ] `grep "\[2.51.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop post-iteration hook: <cmd>` specifies a shell command to run at the end of each iteration
- FR-2: Command runs after auto-commit and after auto-push (if `Loop auto-push: on`), at the very end of the iteration
- FR-3: Exit code 0 = proceed to next iteration; non-zero = HARD STOP
- FR-4: HARD STOP message: `[hands-free] HARD STOP — Post-iteration hook failed (exit <N>): <cmd>. Run /hands-free resume to continue to the next iteration.`
- FR-5: After `/hands-free resume`, the loop continues to the next iteration; the hook is not re-run
- FR-6: The hook runs even if auto-commit is off or no changes were committed this iteration
- FR-7: Command follows cwd-scope classification rules
- FR-8: If absent, no post-iteration hook runs

## Non-Goals

- Multiple post-iteration hooks (single command only)
- Pre-iteration hooks (covered in v2.50.0)
- Retry logic for failed hooks (single attempt per iteration)

## Technical Considerations

- US-001 adds `### Loop Post-Iteration Hook` after `### Loop Pre-Iteration Hook` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- No new code fences needed

## Success Metrics

- Hook timing (after auto-commit/auto-push, end of iteration) documented
- Runs-even-without-commit behavior documented
- Exit code semantics documented
- Resume behavior documented
- Available Persistent Settings table updated
- Even code-fence count, version 2.51.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
