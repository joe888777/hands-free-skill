[PRD]
# PRD: Hands-Free Skill v2.5.0 Cleanup

## Overview

The security automation work (v2.5.0) introduced two overlapping top-level sections in SKILL.md: `## Security Scanning` (lines 2739–2935, added by US-001) and `## Security Automation` (lines 3567+, added by parallel agents). The `## Security Automation` section is the canonical, more complete version. This cleanup consolidates the duplicate, bumps the version to 2.5.0, and adds the CHANGELOG entry.

## Goals

- Remove the duplicate `## Security Scanning` section while preserving any content unique to it
- Verify SKILL.md `code fence` count remains even after removal
- Bump `version:` in SKILL.md frontmatter from `2.4.0` to `2.5.0`
- Add `## [2.5.0]` entry to CHANGELOG.md documenting the security automation feature

## Quality Gates

These commands must pass for every user story:
- Verify SKILL.md markdown validity: `grep -c '^\`\`\`' SKILL.md` must return an even number
- Verify version string updated: `grep "^version:" SKILL.md` must show `2.5.0`
- Verify CHANGELOG entry exists: `grep "\[2.5.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Remove duplicate `## Security Scanning` section

**Description:** As a skill maintainer, I want the duplicate `## Security Scanning` section removed so that readers find one authoritative security reference.

**Acceptance Criteria:**
- [ ] `## Security Scanning` top-level section (starting at line ~2739) is removed from SKILL.md
- [ ] `### Security Scanning` subsection inside `## Auto-Commit` (line ~2639) is retained — it is a subsection header, not a duplicate top-level section
- [ ] `## Security Automation` section (line ~3567) remains intact and unchanged
- [ ] Any content unique to the removed section (not already in `## Security Automation`) is merged into `## Security Automation` before deletion
- [ ] Code fence count (`grep -c '^\`\`\`' SKILL.md`) remains even after removal

### US-002: Bump version and update CHANGELOG

**Description:** As a skill user, I want the version number and changelog to reflect the security automation work so I know what changed in each release.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.4.0` to `2.5.0`
- [ ] New `## [2.5.0] — 2026-03-19` section added at the top of CHANGELOG.md (after the `# Changelog` heading)
- [ ] CHANGELOG entry summarises: pre-commit security scanning, enhanced secrets detection, posture score, `/hands-free security` command, CLAUDE.md overrides, loop integration
- [ ] CHANGELOG entry follows the same format as the existing `## [2.3.0]` entry

## Functional Requirements

- FR-1: After removal, `grep -c "^## Security" SKILL.md` must return `1` (only `## Security Automation` remains)
- FR-2: The `### Security Scanning` subsection inside `## Auto-Commit` must still exist
- FR-3: Version in frontmatter (`version: 2.5.0`) must be on its own line
- FR-4: CHANGELOG `[2.5.0]` entry must appear before `[2.3.0]`

## Non-Goals

- Rewriting or reorganising the `## Security Automation` section content
- Adding new security features
- Changing any other sections of SKILL.md

## Technical Considerations

- The `## Security Scanning` block spans approximately lines 2739–2935; verify exact boundaries with `grep -n` before deletion
- Use Edit tool with the exact first and last lines of the block as old_string
- Check for unique content in the removed block before deleting (compare with `## Security Automation`)

## Success Metrics

- Single authoritative security section in SKILL.md
- Even code-fence count
- Version string reads `2.5.0`
- CHANGELOG entry present and correctly formatted

## Open Questions

- None — scope is fully defined by the existing file state
[/PRD]
