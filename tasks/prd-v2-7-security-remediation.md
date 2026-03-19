[PRD]
# PRD: Hands-Free v2.7.0 — Security Remediation Automation

## Overview

The v2.5.0 security scanning toolkit detects vulnerabilities and grades the workspace but takes no corrective action. This release closes that gap: medium/low vulnerabilities are auto-fixed using the project's native package manager audit fix command; high/critical vulnerabilities block auto-commit and emit a precise, copy-paste-ready fix command; a new `/hands-free security fix` subcommand lets the user trigger remediation on demand; and remediation history is appended to `.claude/security-posture.json` so the loop can track whether fixing is making progress.

## Goals

- Auto-remediate medium/low vulnerabilities without user intervention using `npm audit fix`, `cargo audit fix`, and `pip-audit --fix`
- Surface actionable fix commands for high/critical findings rather than only blocking
- Provide `/hands-free security fix` as an on-demand remediation trigger
- Record remediation attempts (date, scanner, severity fixed, command, outcome) in `.claude/security-posture.json`
- Bump version to 2.7.0 with CHANGELOG entry

## Quality Gates

These commands must pass for every user story:
- `grep -c '^```' SKILL.md` must return an even number
- `grep "^version:" SKILL.md` must show `2.7.0`
- `grep "\[2.7.0\]" CHANGELOG.md` must match

## User Stories

### US-001: Specify auto-fix behavior for medium/low vulnerabilities

**Description:** As a skill user, I want hands-free to automatically run the project's audit fix command for medium and low severity findings so that routine dependency hygiene requires no manual effort.

**Acceptance Criteria:**
- [ ] New `### Auto-Fix Behavior` sub-section added under `## Security Automation`
- [ ] Specifies per-scanner auto-fix commands: `npm audit fix` (Node), `cargo audit fix` (Rust), `pip-audit --fix` (Python), `bundle update --conservative` (Ruby)
- [ ] Specifies the two-phase approach: first run `--dry-run` variant, check exit code; if safe, run without `--dry-run`
- [ ] Auto-fix only triggers when finding severity is **medium or low** — not high or critical
- [ ] After auto-fix, re-run the scanner to confirm the finding is resolved; if still present after fix, log as warning and continue (do not block)
- [ ] Auto-fix announce format: `[security] Auto-fixed N medium/low findings via <command>`
- [ ] Auto-fix is skipped if the dry-run variant is not supported by the scanner; in that case, skip auto-fix and treat as high severity (emit fix command, do not auto-apply)
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-002: Specify actionable fix commands for high/critical vulnerabilities

**Description:** As a skill user, I want hands-free to emit a precise fix command when high/critical vulnerabilities block auto-commit so that I know exactly what to run to resolve the block.

**Acceptance Criteria:**
- [ ] New `### High/Critical Fix Commands` sub-section added under `## Security Automation`
- [ ] When a high/critical finding blocks auto-commit, emit the specific fix command as part of the HARD STOP message
- [ ] Fix command format per scanner: `npm audit fix --force` (Node), `cargo update -p <crate>` (Rust), `pip install --upgrade <package>==<safe-version>` (Python), `bundle update <gem>` (Ruby)
- [ ] If no safe version available: emit `[security] No automated fix available for <finding> — manual review required`
- [ ] HARD STOP message format: `[security] Auto-commit blocked — critical: <finding>. Run: <fix-command>`
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-003: Add `/hands-free security fix` subcommand

**Description:** As a skill user, I want a `/hands-free security fix` command that triggers on-demand remediation so I can fix vulnerabilities outside of the auto-commit flow.

**Acceptance Criteria:**
- [ ] `/hands-free security fix` added to the Commands code block in SKILL.md
- [ ] `/hands-free security fix` added to the Quick Reference table in SKILL.md
- [ ] New sub-section documenting `/hands-free security fix` behavior added under `## Security Automation`
- [ ] Command detects project type, runs appropriate scanner, applies auto-fix for medium/low, emits fix commands for high/critical
- [ ] Command reports outcome: `[security] Fix complete — N findings resolved, M findings require manual review`
- [ ] Command re-runs scanner after fixes and updates `.claude/security-posture.json` with new grade
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-004: Specify remediation history tracking in security-posture.json

**Description:** As a skill user, I want remediation attempts recorded in `.claude/security-posture.json` so I can see whether repeated loop iterations are making progress.

**Acceptance Criteria:**
- [ ] New `### Remediation History` sub-section added under `## Security Automation`
- [ ] Specifies `remediation_history` array field in `.claude/security-posture.json`
- [ ] Each entry contains: `date` (ISO 8601), `scanner`, `severity_fixed` (array), `command`, `outcome` (`resolved`/`partial`/`failed`)
- [ ] Maximum 20 entries retained (oldest pruned when limit exceeded)
- [ ] JSON schema example included
- [ ] Loop integration: if last 3 attempts for same scanner all `outcome: failed` → route to systematic-debugging
- [ ] `grep -c '^```' SKILL.md` returns even number

### US-005: Bump version to 2.7.0 and update CHANGELOG

**Description:** As a skill user, I want the version and changelog to reflect the security remediation feature.

**Acceptance Criteria:**
- [ ] `version:` field in SKILL.md frontmatter changed from `2.6.0` to `2.7.0`
- [ ] New `## [2.7.0] — 2026-03-19` section added at top of CHANGELOG.md
- [ ] CHANGELOG entry summarises: auto-fix for medium/low, fix commands for high/critical, `/hands-free security fix`, remediation history
- [ ] `grep "^version:" SKILL.md` shows `2.7.0`
- [ ] `grep "\[2.7.0\]" CHANGELOG.md` matches

## Functional Requirements

- FR-1: Auto-fix must use dry-run variant when available; skip if dry-run shows breaking changes
- FR-2: Auto-fix only applies to medium and low findings; high/critical always require explicit action
- FR-3: After auto-fix, re-run scanner to confirm resolution
- FR-4: `/hands-free security fix` works in all modes when explicitly invoked
- FR-5: In crazy-workspace, fix applies including for high severity (local package files only)
- FR-6: Remediation history is append-only; cap at 20 entries (FIFO)
- FR-7: Fix commands in HARD STOP messages must be copy-paste-ready
- FR-8: If no scanner installed: `[security] No scanner available for <type> — install <scanner> to enable auto-fix`

## Non-Goals

- Automatically applying high/critical fixes without user confirmation (outside crazy-workspace)
- Patching source code vulnerabilities
- CVE research beyond what the scanner provides
- Fixing vulnerabilities where no fix is available in the package manager

## Technical Considerations

- All edits to `## Security Automation` section (distinct sub-sections)
- US-003 also modifies Commands code block (~line 58) and Quick Reference table (~line 28)
- US-005 modifies frontmatter and CHANGELOG.md
- US-004 JSON schema example uses fenced code block with `json` language tag

## Success Metrics

- Auto-fix behavior specified with per-scanner commands
- HARD STOP includes copy-paste fix command
- `/hands-free security fix` in Commands, Quick Reference, and dedicated sub-section
- `remediation_history` schema example present
- Even code-fence count, version 2.7.0, CHANGELOG entry

## Open Questions

- None
[/PRD]
