# Changelog

## [2.3.0] — 2026-03-17

### Added (iteration 6–7)

**New command**
- `/hands-free check <command>` — preview command classification (auto-pass / ask / HARD STOP) without running it; shows matched rule and safe alternative for hard stops
- Quick Reference table updated with `/hands-free check`

**Security**
- Shell script content scan: Edit/Write tool scans script content for hard stop patterns before writing; applies in all modes including crazy-workspace
- Subshell `$(...)` rule: `bash $(curl URL)` → HARD STOP; inner command's classification propagates to outer
- Complex shell constructs (`if/for/while/case`): classified by most restrictive branch
- Heredoc pattern rule: local DB heredoc → auto-pass; remote DB or POST heredoc → ask
- Secrets expanded: Slack tokens (`xoxb-`, `xoxp-`), PGP key marker, `client_secret=`, `totp_secret=`, `smtp_password=`, `ftp_password=`, `sftp_password=`, `access_key=`, `auth_token=`, `X-Auth-Token:` header, connection string passwords
- Security Philosophy section explaining WHY hard stops exist

**Tool coverage — Python**
- `uv sync`, `uv add`, `uv remove`, `uv venv`, `uv pip compile`, `uv tool run` → auto-pass
- `poetry install/add/run`, `pipenv install/run` → auto-pass
- `black`, `isort`, `bandit`, `safety`, `coverage run/html`, `pytest --cov` → auto-pass
- `hatch build/run`, `python -m build`, `flit build` → auto-pass

**Tool coverage — TypeScript/Frontend**
- `tsup`, `vite build/dev`, `esbuild`, `rollup` → auto-pass (bundlers)
- `biome check/format`, `ts-node`, `tsx`, `npx prettier --write/--check` → auto-pass
- `vitest watch`, `jest --watchAll` → auto-pass

**Tool coverage — Rust**
- `cargo nextest run`, `cargo expand`, `cargo fix`, `cargo clippy --fix`, `cross build`, `cargo miri test` → auto-pass
- `cargo watch -x run/test`, `cargo outdated`, `cargo tree`, `cargo deny check`, `cargo machete` → auto-pass
- `cargo install --path .` → ask (writes to `~/.cargo/bin`)

**Tool coverage — Build systems**
- `cmake -B build -S .`, `cmake --build build`, `ninja -C build`, `meson setup/compile` → auto-pass
- `make clean/all/lint/fmt/check` → auto-pass; `make install/uninstall` → ask

**Tool coverage — Services**
- `uvicorn/gunicorn` localhost → auto-pass; `--host 0.0.0.0` → ask
- `flask run`, `python manage.py runserver` → auto-pass
- `pm2 list` → auto-pass; `pm2 start/stop/restart` → ask
- `act`, `circleci local execute` → auto-pass (local CI runners)

**Tool coverage — Docker**
- `docker cp`, `docker logs`, `docker inspect`, `docker pull` → auto-pass
- `docker run --rm` with well-known image → auto-pass; unfamiliar image → ask
- `docker buildx build` → auto-pass

**Tool coverage — Redis**
- `redis-cli` read ops on localhost → auto-pass; write/destructive ops and remote hosts → ask

**Tool coverage — SQL DDL**
- `psql -c "DROP TABLE"` / `TRUNCATE` → ask even on local DB
- `psql -c "SELECT"` / `INSERT` / `CREATE TABLE` (local) → auto-pass
- `sqlite3 ./db "INSERT"` → auto-pass; `sqlite3 ./db "DROP TABLE"` → ask

**Git**
- `git pull`, `git pull --rebase` → auto-pass in full; ask in partial
- `git pull --ff-only` → auto-pass (safe fast-forward)
- `git rebase --continue/skip/abort`, `git cherry-pick --continue/abort`, `git merge --abort` → auto-pass
- `git apply ./patch.diff`, `git am ./patches/` → auto-pass (local patches)
- `git stash drop`, `git stash clear` → ask (irreversible)
- `git ls-files`, `git blame`, `git shortlog`, `git describe` → auto-pass

**Shell rules**
- System inspection commands: `ps aux`, `lsof`, `netstat`, `ss`, `df`, `du` → auto-pass (read-only)
- `jq`, `awk`, `sed`, `xargs`, `sort`, `uniq`, `head`, `tail`, `wc`, `tee` (cwd-scoped) → auto-pass
- `wget -O ./file URL` → auto-pass (downloads to cwd); `wget -O /outside URL` → ask
- `curl -s URL > ./file.json` → auto-pass (GET to cwd); `> /tmp/file` → ask
- `$CARGO_HOME`, `$GOPATH`, `$RUSTUP_HOME`, `$GOROOT` added to shell variable escape list
- Package updates: `cargo update`, `npm update`, `pnpm update`, `yarn upgrade` → auto-pass
- `pip install --upgrade <pkg>` (venv active) → auto-pass; (no venv) → ask
- `npm run <script>`: known-safe targets auto-pass; unknown/deploy targets ask
- `npx <well-known>` → auto-pass; `npx <unknown>@latest` → ask
- `git stash drop` / `git stash clear` → ask
- `cargo generate --git <url>` → ask; `cargo generate <local>` → auto-pass
- `./script.sh` / `bash ./script.sh` (cwd-scoped, known-safe content) → auto-pass

**Auto-commit edge cases**
- Detached HEAD → skip with announcement
- Bare repository → skip silently
- Staged files from previous task (not modified by Claude) → exclude from auto-commit

**CLAUDE.md behavior**
- User `/hands-free` commands override CLAUDE.md directives for the current session
- Project security rules in CLAUDE.md cannot be overridden by user commands
- `/hands-free status` shows active CLAUDE.md overrides

**Troubleshooting**
- "Auto-commit skipped in detached HEAD": create a branch first
- "Migration revert blocked": expected behavior; add CLAUDE.md override for dev databases
- "Redis commands blocked unexpectedly": add CLAUDE.md override for non-standard local hostnames
- `cargo install --path .` ask: expected; writes to `~/.cargo/bin`, not cwd

## [2.2.0] — 2026-03-17

### Added (iteration 5)

**Security**
- `deno run https://...` → universal HARD STOP (language-level RCE — Deno natively fetches and runs URLs)
- `perl` fetch-then-eval patterns → universal HARD STOP (added to language RCE list)
- `docker compose push` → shared/remote state hard stop (pushes to external registry)
- `terraform apply`, `terraform destroy` → shared/remote state hard stop (modifies external infrastructure)
- `ssh`, `scp`, `rsync` to remote hosts → ask (remote machine access, not within `./`)

**Secrets Detection**
- Filename patterns: `.npmrc` (npm auth tokens), `*.cer`, `*.der`, `*.crt` (certificate files)
- Content signals: `database_url=`, `signing_key=` assignment patterns
- Content signals: hardcoded `Authorization: Bearer ` and `X-Api-Key:` HTTP headers in source files

**Shell Auto-Pass**
- `git tag <name>` → always-auto-pass (local tag creation, non-destructive)
- `git commit -m "..."` (non-amend, no `-a` flag) → always-auto-pass
- `git worktree add <path>` → always-auto-pass; `git worktree remove` is NOT auto-pass
- `git submodule update --init [--recursive]` → always-auto-pass
- `git submodule add <url>` → auto-pass in full, ask in partial (execution-type)
- `docker compose up/build/down/run` → auto-pass; `docker compose push` → ask
- `bun install` → auto-pass (equivalent to npm install)
- `make test`, `make build` → auto-pass; `make install` → ask (may write to system paths)
- `pre-commit run --all-files` → auto-pass (runs hooks locally)
- `terraform plan` → auto-pass (read-only dry run)

**Rules**
- Env-var prefix rule: `KEY=value cmd` classified by underlying `cmd`, not by the env var
- Execution-type vs design-type for custom skills in partial mode: explicit classification criteria
- Remote DB example fix: `psql postgresql://prod-db/mydb -c "DROP TABLE"` → ask (not HARD STOP; consistent with remote DB rule)

**Mode Behavior Table**
- Added "Language RCE" row (separate from curl|bash pipe-to-shell row)

**Documentation**
- README FAQ: 3 new entries (docker compose push vs up, Claude Code permission system interaction, partial mode execution-type)
- Troubleshooting: `deno run` local vs URL clarification; terraform apply/destroy
- README security table updated with deno run URL

## [2.1.0] — 2026-03-17

### Added (iteration 4 — continued from 2.0.0)

**Security**
- `sudo -s`, `sudo su`, `sudo bash`, `sudo sh` (interactive root shell) → universal HARD STOP
- Global package install detection: `npm install -g`, `cargo install`, `pip install` without venv → ask (writes outside cwd)
- Docker volume mount rules: `-v ./:/app` auto-pass; `-v /:/host`, `-v ~/.ssh:/ssh` ask
- `git config --global/--system` → ask (modifies global git config)

**UX / Commands**
- `/hands-free` with no argument: activate full mode if not active; show status if already active
- `/hands-free learning` with no argument: show current level and threshold summary
- Mode persistence guidance: explains session-scoped reset; how to persist via CLAUDE.md
- Announcement format table: maps each decision source to its output format (silent, announce, hard stop)
- Version bumped to 2.1.0; frontmatter description updated to include language-level RCE and global installs

**Shell auto-pass**
- `nvm`, `rustup` added to always-auto-pass list
- `pnpm install`, `yarn install` added to always-auto-pass list
- Remote DB connection string detection: `postgresql://non-localhost` in command line → ask
- Database migration examples: `sqlx migrate run`, `alembic upgrade head`, `npx prisma migrate dev`, `diesel migration run`
- Additional examples: `pnpm run build`, `yarn test`, `python -m venv .venv`, `docker run -v ./`, `sudo -s`

**Custom skill integration**
- Implicit recommendation detection: "I recommend...", "I suggest...", "best option is..." → treated as explicit recommendation
- Custom skill's own confirmation prompts handled as standard checkpoint approvals

**Ralph Loop**
- Loop stall prevention: 3 iterations with no new progress → stall warning and mandatory pause
- Explicit note: does NOT output completion promise unless condition is genuinely true

**Auto-commit**
- Merge conflict detection blocks auto-commit when conflicts are unresolved
- `git commit --amend`, `git commit -a`, `git add -A` added to forbidden operations
- Merge conflict troubleshooting entry added

**`/hands-free explain`**
- Extended to explain hard stop decisions (not just auto-accept decisions)
- Shows matched pattern, rule tier, and safe alternative when applicable

**Review Checkpoints**
- [R] Revise option behavior clarified: prompts user to describe revision, applies it, then re-surfaces checkpoint
- [S] Stop option announces waiting state and expected resume instruction

**Shell auto-pass (continued)**
- `git checkout -b`, `git branch`, `git stash/pop`, `git log/status/diff` added to always-auto-pass list
- `python manage.py migrate`, `python manage.py makemigrations` added (Django)
- `cargo publish`, `npm publish`, `docker push`, `vercel deploy`, `zeabur deploy` added as ask (external registries/services)
- Publish/deploy commands added to shared/remote state hard stop list

**Security (continued)**
- Deployment/publish keyword detection in custom skill approvals: "deploy", "publish", "push to [service]" keywords trigger hard stop pattern recognition
- Partial git add failure: announce which files failed before attempting partial commit

**Reliability**
- `preferences.md` corruption: continue with defaults, announce once
- `/hands-free status` in off mode: shows learning level and preferences loaded count
- Optional review checkpoints silently ignored in partial mode (always on, not user-configurable)
- Single-option presentation = confirmation → auto-accept in full mode

**Superpowers integration**
- `test-driven-development` approval points added to table
- `verification-before-completion` routing documented: auto-verify → debugging on failure
- Mode switch announcements defined for all transitions (including off→full, off→partial)
- Agent tool dispatch guidance: full mode auto-approves, partial mode asks for execution-type agents

## [2.0.0] — 2026-03-17

### Added (iteration 3 — same day)

**Security**
- Language-level RCE hard stops (universal): `python -c "exec(urllib...)"`, `node -e "eval(...)"`, `ruby -e "eval(URI.open...)"`, `source <(curl ...)` blocked in ALL modes
- Updated Quick Reference and README security table to include all RCE variants and language-specific patterns

**Auto-commit**
- Merge conflict detection: skip auto-commit and announce when unresolved conflicts exist in working tree
- Added `git commit --amend` and `git commit -a` / `git add -A` to the forbidden operations list
- Updated edge cases table with merge conflict scenario

**Learning / Preferences**
- `/hands-free learning` with no argument: show current learning level and threshold summary
- Sensitivity vs. confidence clarification: explains that confidence tier = permanence in preferences.md; sensitivity = how aggressively current session applies it
- Preference pruning guidance: when and how to remove stale observations from preferences.md

**Shell examples**
- TypeScript: `npx eslint src/`, `npx vitest run`, `bun run test`, `bun run build`
- Python: `python -m mypy src/`, `python -m ruff check .`, `uv run pytest`
- Rust: `cargo fmt`, `cargo fmt --check`, `cargo sqlx prepare`
- New HARD STOP examples: `source <(curl ...)`, `python -c "exec(urllib...)"`, `node -e "eval(...)"`

**Superpowers integration**
- Added `dispatching-parallel-agents` and `requesting-code-review` to Superpowers-Specific Approval Points table

**Ralph Loop**
- Loop stall prevention: detect 3 consecutive iterations with no new progress; announce stall warning and pause for user input
- Clarified: does NOT output completion promise unless condition is genuinely true

**UX**
- Mode table note: optional review checkpoints become HARD STOP when `review-checkpoints on` is set
- `off` mode clarification: governs skill-level approval points, not Claude Code tool execution system
- Improved dry-run output: shows mode-specific state for optional checkpoints and rm-rf patterns
- Write-Capable Tool Rules section: documents Edit/Write/NotebookEdit auto-pass rules and secrets scan behavior

### Added

**Security**
- Pipe-to-shell hard stop (universal): `curl | bash`, `wget | sh`, `eval $(curl ...)`, `bash <(curl ...)` blocked in ALL modes including crazy-workspace
- Privilege escalation hard stop (universal): `chmod 777`, `chmod -R 777`, `sudo` to system paths blocked in ALL modes
- Secrets detection before every auto-commit: filename patterns (`.env`, `*.pem`, `*.key`, etc.) and content signals (`sk-`, `ghp_`, `AKIA`, `-----BEGIN RSA`, `password=`, etc.); applies in all modes including crazy-workspace; cannot be overridden
- Path traversal clarification: symlink escapes, `..`-normalized paths, `$HOME`/`~`/`$XDG_*` expansions that exit cwd now block shell auto-pass

**Automation**
- `/hands-free dry-run`: preview what would be auto-accepted without enabling hands-free or changing state
- `/hands-free pause` / `/hands-free resume`: temporarily suspend auto-accept without changing mode; paused state shown in status
- `/hands-free explain`: trace the reasoning behind the most recent auto-accept decision
- `/hands-free reset`: clear all learned preferences with confirmation prompt (always pauses, never auto-accepted)
- `/hands-free review-checkpoints on/off`: structured pause-and-summarize before costly phase transitions; two mandatory checkpoints (before execution, before push/merge)
- Iteration warnings in ralph-loop: print at 3 and 2 remaining iterations; mandatory PAUSE at 1 remaining
- Git-log-based loop state detection: concrete algorithm using `git log --oneline -20` + `git status --short`; five-state decision table replaces heuristic prose

**UX / Documentation**
- Quick Reference table at top of skill: maps user intent to commands; includes "Always blocked" reminder
- `/hands-free status`: fully defined with mode, learning, auto-commit, review checkpoints, pause state, loop-aware state, session decisions, and tiered hard stop lists
- Recommended Setup: expanded from single line to use-case table covering 5 scenarios
- Mode transitions: defined mid-session switching behavior; `partial` auto-enables review-checkpoints
- Conflict resolution: priority rules when two skills present simultaneous approval points
- Session log: added 6 new event types (`[review-checkpoint]`, `[auto-commit]`, `[verification]`, `[hard-stop]`, `[user-override]`)
- "When There Is No Recommended Option": pick learned preference, then first-listed, then log and continue
- Custom Skill Integration: documented how hands-free recognizes approval patterns in non-superpowers skills
- Preference staleness: downgrade rules when user repeatedly contradicts a learned preference
- Preferences scoping: global (not per-project); added "What NOT to record" guidance
- Auto-commit edge cases: 6 scenarios (no changes, no git repo, hook failure, secrets in staged files, git add failure, untracked-only)
- Shell examples: added `cargo build`, `make build` (auto-pass); `curl|bash`, `wget|sh`, `eval$(curl)`, `chmod 777`, `sudo /etc/` (HARD STOP)

### Fixed

- Crazy-workspace activation announcement: now lists all 5+ hard stops (was listing only `rm -rf *` and `rm -rf .git`)
- Crazy-workspace "Two absolute hard stops": updated to "Absolute hard stops (no exceptions, no override)"
- Crazy-workspace decision flowchart: updated to check pipe-to-shell, chmod 777, secrets in addition to rm -rf patterns
- HARD STOP section: restructured into two-tier (universal vs standard); git push/merge now annotated as crazy-workspace overrideable within `./`; removed duplicate "crazy-workspace too" block
- Status output: `git push` moved from flat "Active hard stops" to "Mode hard stops"; `secrets-in-commit` and `Paused:` field added
- Red flags table: clarified mode-specific vs universal stops; added "crazy-workspace allows everything" misconception row
- Command comment for `crazy-workspace`: updated from "except rm -rf * and rm -rf .git" to "5 universal hard stops remain"
- `/hands-free recommend` section: added explicit "will NEVER suggest" rule for universal hard stops
- Frontmatter description: updated to reflect all current capabilities (was stale, mentioned only 3 modes)

### Changed

- Mode behavior table: added `curl | bash`, `chmod 777`, secrets, and review checkpoint rows (19 rows total, up from 10)
- Auto-commit safety rules: expanded from single bullet to full secrets detection procedure with filename/content patterns
- Shell auto-pass rules: added symlink escapes, normalized `..` paths, `$HOME`/`~`/`$XDG_*` as path-escape conditions
- README mode table: expanded from 5 rows to 13 rows; added crazy-workspace column; includes all new hard stop patterns
- README commands section: all 16 commands documented including dry-run, pause/resume, explain, reset, review-checkpoints
- README "What it does": added pipe-to-shell/secrets blocking and review checkpoints; updated "Three modes" to "Four modes"

### Added (continued, same day — iteration 2)

- Read-only tool auto-pass: Grep, Glob, Read, WebFetch/WebSearch explicitly documented as always auto-approved (read-only, no side effects); mode table row updated from "Grep tool calls" to "Read-only tools"
- CLAUDE.md integration: per-project preference overrides via CLAUDE.md `# hands-free overrides` section; takes precedence over global preferences.md
- FAQ section in README: 6 common questions (custom skill compatibility, safety, preference updates, crazy-workspace on shared repos, sensitive data, disabling learning)
- Shadow mode: `/hands-free off` + `learning high` added to Recommended Setup table
- Auto-commit: step 2 now explicitly reads `git log --oneline -5` to detect commit message style
- Crazy-workspace production warning: blockquote warning not to use on production repos
- Comprehensive decision flowchart: updated digraph in HARD STOP section to show full path through universal hard stops → paused/off → review checkpoint → learned preference → mode allows
- Session log: documented as in-memory only (not persisted), with pointer to git log and preferences.md for durability
- Loop state algorithm: fallback documented when user commits without auto-commit (no `[ralph #N]` tags)
- `What's new in 2.0` section in README

### Fixed (same day — iteration 2)

- `rm -rf *`: README mode table showed HARD STOP in all modes — corrected to `ask` in full/partial/off (user is prompted, not blocked); HARD STOP only in crazy-workspace
- `git push`: README mode table showed HARD STOP in full/partial/off — corrected to `ask` (user can confirm); semantics: `ask` = pause+confirm, HARD STOP = blocked
- Shared/remote state: incorrectly annotated as crazy-workspace overrideable — external services (GitHub API, Slack) are not within `./` so crazy-workspace does not override
- Crazy-workspace behavior description: "CI changes" → "CI/CD workflow file edits" to distinguish local file edits (auto-approved) from triggering external deployments (hard stop)
- Quick Reference table: `/hands-free log` and `/hands-free recommend` were missing
- Recording format: `## Learned Rules` split into high/medium sections to match preferences.md structure

### Added (continued, later in same day)  (original list)

- `version: 2.0.0` in frontmatter
- Troubleshooting section: 5 scenarios (not auto-accepting, blocking unexpectedly, preferences not applying, auto-commit unexpected files, loop exhausted)
- `/hands-free recommend`: expanded output with override stats, auto-commit suggestion, review-checkpoint trigger; smart suggestion thresholds; `/hands-free recommend promote <action>` subcommand
- `/hands-free explain`: trace the reasoning behind any auto-accept decision
- `/hands-free pause` / `/hands-free resume`: temporarily suspend without changing mode
- `/hands-free reset`: clear all learned preferences with confirmation
- Session Log: persist note (in-memory only); all 10 event types listed
- Mode transitions: defined mid-session switching; `partial` auto-enables review-checkpoints
- Conflict resolution: priority rules for competing skill approval points
- "When There Is No Recommended Option": fall back to preference → first-listed → log
- Custom Skill Integration: recognition patterns for non-superpowers skills
- Preferences scoping, staleness, and "What NOT to record" rules
- Ralph loop state file edge cases (missing file, null max_iterations, malformed YAML)
- Iteration-aware commit fallback when `[ralph #N]` tags absent
- Iteration warnings (print at 3/2 remaining, PAUSE at 1 remaining)
- Loop state algorithm fallback note when user commits without auto-commit
- Auto-commit edge cases: 6 scenarios
- Shell examples: `cargo build`/`make build`/`curl -o ./tool` (auto-pass); `curl|bash`/`chmod 777`/`sudo /etc/` (HARD STOP)
- Quick Reference table at top of skill

## [1.0.0] — 2026-03-16

### Changed

- **crazy-workspace mode**: Simplified behavior description — auto-approve all operations within `./` without the redundant qualifier. Hard stops remain: `rm -rf *` and `rm -rf .git`.
