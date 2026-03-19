[PRD]
# PRD: Hands-Free v2.45.0 — Loop Commit Message Prefix

## Overview

Auto-commits in loop mode are currently tagged with `[ralph #N]` where N is the iteration number. For projects with different commit message conventions or CI systems that filter on specific prefixes, this hardcoded tag may not fit. This release adds a `Loop commit prefix: <prefix>` directive that replaces the default `[ralph #N]` prefix with a user-specified string.

## Goals

- Add `Loop commit prefix: <prefix>` CLAUDE.md directive
- When set, use `<prefix> feat: <description>` instead of `[ralph #N] feat: <description>` for auto-commits in loop mode
- Iteration number is still appended after the prefix if it contains `{N}` placeholder (e.g., `[loop-{N}]` → `[loop-3]`)
- Add directive to Available Persistent Settings table
- Bump version to 2.45.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.45.0`
- `grep "\[2.45.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Document loop commit prefix in Ralph Loop Integration

**Description:** As a skill user, I want auto-commit messages to use my project's commit prefix convention so the loop's commits fit naturally into the git log.

**Acceptance Criteria:**
- [ ] New `### Loop Commit Prefix` sub-section added under `## Ralph Loop Integration` (after `### Loop Test Command`, before `### What Hands-Free Does NOT Do in Loop Mode`)
- [ ] Documents that the prefix replaces `[ralph #N]` in auto-commit messages
- [ ] Documents `{N}` placeholder: if prefix contains `{N}`, it is replaced with the current iteration number
- [ ] Examples provided: `Loop commit prefix: [ci]` → `[ci] feat: ...`; `Loop commit prefix: iter-{N}:` → `iter-3: feat: ...`
- [ ] Documents that the prefix applies only to auto-commits made during loop mode; manual commits and non-loop auto-commits are unaffected
- [ ] If absent, default `[ralph #N]` prefix is used
- [ ] Default: absent — `[ralph #N]` prefix used
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Add CLAUDE.md override for loop commit prefix

**Description:** As a skill user, I want to configure the commit prefix in CLAUDE.md.

**Acceptance Criteria:**
- [ ] `Loop commit prefix: <prefix>` row added to Available Persistent Settings table
- [ ] Effect description notes the `{N}` placeholder and that it replaces `[ralph #N]`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Bump version to 2.45.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the commit prefix feature.

**Acceptance Criteria:**
- [ ] `version:` field changed from `2.44.0` to `2.45.0`
- [ ] New `## [2.45.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: commit prefix override, {N} placeholder, replaces [ralph #N]
- [ ] `grep "^version:" SKILL.md` shows `2.45.0`
- [ ] `grep "\[2.45.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: `Loop commit prefix: <prefix>` replaces `[ralph #N]` as the leading tag in auto-commit messages during loop mode
- FR-2: If the prefix contains `{N}`, replace `{N}` with the current iteration number at commit time
- FR-3: The prefix is prepended to the commit message exactly as specified (no additional space is added between prefix and message unless the prefix ends with a space)
- FR-4: Applies only to auto-commits made during loop mode; non-loop auto-commits are unaffected
- FR-5: If absent, default `[ralph #N]` prefix behavior is unchanged

## Non-Goals

- Changing the commit message body (only the prefix/tag is affected)
- Per-iteration custom messages (prefix is a static template with optional {N})
- Removing the prefix entirely (use an empty string or a single space if no prefix is desired)

## Technical Considerations

- US-001 adds `### Loop Commit Prefix` after `### Loop Test Command` in `## Ralph Loop Integration`
- US-002 adds one row to Available Persistent Settings table
- US-003 modifies frontmatter and CHANGELOG.md
- No new code fences needed

## Success Metrics

- Prefix replacement and {N} placeholder documented with examples
- Scope (loop mode only) clear
- Available Persistent Settings table updated
- Even code-fence count, version 2.45.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
