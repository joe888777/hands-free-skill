[PRD]
# PRD: Hands-Free v2.8.0 — Iteration Context Checkpoint

## Overview

Each ralph-loop iteration starts with no memory of prior work. This forces the loop to re-brainstorm already-decided designs and re-detect already-completed work. This release adds a structured `.claude/iteration-checkpoint.json` file that is written at the end of each iteration and read at the start of the next, giving the loop rich context without re-brainstorming. A `/hands-free context` command exposes the checkpoint on demand.

## Goals

- Write a structured checkpoint at iteration end recording what was accomplished, last commit, test status, and pending plan items
- Read checkpoint at next iteration start and include a rich context block in the iteration announcement
- Prevent re-brainstorming when an approved design already exists from a prior iteration
- Provide `/hands-free context` as a read-only on-demand view of the checkpoint
- Guard against stale checkpoints (sha not in git log)
- Bump version to 2.8.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.8.0`
- `grep "\[2.8.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Specify checkpoint write behavior at iteration end

**Description:** As a loop operator, I want a checkpoint file written at the end of each iteration so that subsequent iterations start with full context.

**Acceptance Criteria:**
- [ ] New `### Iteration Context Checkpoint` sub-section added under `## Ralph Loop Integration`
- [ ] Specifies that `.claude/iteration-checkpoint.json` is written (or overwritten) at the end of every loop iteration
- [ ] Documents the JSON schema: `iteration` (int), `last_commit_sha` (string), `last_commit_message` (string), `completed_stories` (array of strings), `pending_stories` (array of strings), `test_summary` (`{ passed: int, failed: int, last_run: ISO8601 }`), `security_grade` (string or null), `active_plan_file` (string or null), `written_at` (ISO8601)
- [ ] Documents that the file is written AFTER auto-commit (so `last_commit_sha` reflects the iteration's committed work)
- [ ] Documents that the file is auto-added to `.gitignore` (not committed — it is session state)
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Specify checkpoint read and enhanced iteration announcement

**Description:** As a loop operator, I want the iteration start announcement to include context from the previous iteration's checkpoint so I can see what was done and what is next without re-reading git log.

**Acceptance Criteria:**
- [ ] `### Loop-Aware Behavior` section updated: at iteration start, if `.claude/iteration-checkpoint.json` exists, read it and include a context block in the announcement
- [ ] Stale checkpoint guard documented: if `last_commit_sha` not in `git log --oneline -20`, warn and include checkpoint anyway
- [ ] Enhanced announcement format documented with sample output showing last commit, completed/pending stories, tests, security grade
- [ ] If no checkpoint exists (iteration 1 or missing file): fall back to existing announcement format
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Add `/hands-free context` command

**Description:** As a loop operator, I want a `/hands-free context` command that prints the checkpoint summary on demand.

**Acceptance Criteria:**
- [ ] `/hands-free context` added to the Commands code block in SKILL.md
- [ ] `/hands-free context` added to the Quick Reference table in SKILL.md
- [ ] New `## /hands-free context` section added to SKILL.md (after `## /hands-free health`)
- [ ] Section documents output format and when-no-checkpoint message
- [ ] Command is read-only
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-004: Bump version to 2.8.0 and add CHANGELOG entry

**Description:** As a skill user, I want the version and changelog to reflect the iteration context checkpoint feature.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.7.0` to `2.8.0`
- [ ] New `## [2.8.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: iteration checkpoint write, enhanced announcement, `/hands-free context`, stale-sha guard
- [ ] `grep "^version:" SKILL.md` shows `2.8.0`
- [ ] `grep "\[2.8.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: Checkpoint written after auto-commit completes (sha is final)
- FR-2: Checkpoint NOT committed to git (auto-gitignore)
- FR-3: Enhanced announcement only shown when checkpoint exists and is readable
- FR-4: Stale sha warning is advisory — does not block the iteration
- FR-5: `/hands-free context` works in all modes including off
- FR-6: `pending_stories` derived from active plan file if it exists; otherwise empty array
- FR-7: Malformed checkpoint: skip and fall back to plain announcement with log message

## Non-Goals

- Persisting checkpoint across git branches
- Trend analysis across multiple checkpoints
- Automatic plan file detection beyond PLAN.md and .claude/plan.md

## Technical Considerations

- US-001 adds new sub-section to Ralph Loop Integration
- US-002 modifies existing Loop-Aware Behavior sub-section
- US-003 modifies Commands block, Quick Reference, and adds a new top-level section after /hands-free health
- US-004 modifies frontmatter and CHANGELOG.md only
- JSON schema example uses fenced code block with json language tag

## Success Metrics

- `.claude/iteration-checkpoint.json` schema documented with JSON example
- Enhanced announcement format documented with sample output
- `/hands-free context` in Commands, Quick Reference, and dedicated section
- Even code-fence count, version 2.8.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
