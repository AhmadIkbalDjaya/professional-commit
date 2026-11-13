# Professional Commit

Agent skill for safe, atomic, professional Git commits. It audits working tree and index, learns repository commit conventions, proposes a complete commit plan, waits for explicit approval, then stages exact approved paths and verifies each commit.

Built with open [Agent Skills](https://agentskills.io) format.

## Default flow

```text
Audit → plan → explicit approval → stage approved files → commit → verify
```

No files are staged or committed during audit. No push occurs.

## Features

- Reads repository guidance, commitlint configuration, hooks, and recent Git history.
- Uses English for every commit subject, body, and footer.
- Follows project convention; defaults to Conventional Commits when no local standard exists.
- Separates staged, unstaged, and untracked changes before proposing scope.
- Plans atomic commits and flags unrelated changes for separate commits.
- Generates exact commit messages, including scopes, body bullets, breaking-change notes, and issue footers when needed.
- Uses bullet body for non-trivial commits with multiple changes, effects, rationale, or validation facts.
- Runs available, relevant checks: whitespace, tests, type checks, lint, format, and build when required.
- Detects likely secrets, private keys, conflict markers, unexpected binary/large files, generated output, and suspicious lockfile changes.
- Stages exact approved paths only. Never defaults to `git add .` or `git add -A`.
- Keeps Git hooks enabled and verifies created commit with `git show`.

## Commit message format

Default format when repository has no established alternative:

```text
<type>(<scope>)!: <subject>

- <change or impact one>
- <change or impact two>
- <test, migration, or important effect>

BREAKING CHANGE: <required migration detail>
Closes #<issue>
```

Optional parts are omitted when empty.

### Types

| Type | Use |
|---|---|
| `feat` | New user-facing capability |
| `fix` | Bug correction |
| `refactor` | Internal change without behavior change |
| `perf` | Performance improvement |
| `docs` | Documentation only |
| `test` | Test-only change |
| `build` | Build system or dependencies |
| `ci` | Continuous integration configuration |
| `chore` | Maintenance outside product code |
| `style` | Formatting-only change |
| `revert` | Reverts prior commit |

### Header rules

- English, imperative, lowercase, specific, no final period.
- Target subject length: 50 characters or fewer.
- Header limit: 72 characters unless repository convention differs.
- Scope is optional. Use only stable, useful names such as `auth`, `api`, `checkout`, or package name.
- Avoid vague subjects: `update`, `changes`, `misc`, `fix stuff`, `WIP`.

### Bullet body rules

No body needed for a small, self-explanatory change. Use body bullets for a non-trivial commit.

- One bullet per meaningful change, impact, rationale, test, or migration fact.
- Start bullets with verbs: `Add`, `Update`, `Remove`, `Prevent`, `Refactor`, `Cover`, `Document`.
- Keep lines near 72 characters.
- Explain why when it adds review value. Do not restate subject.
- Never state tests passed unless skill ran them successfully.

Example:

```text
feat(auth): rotate refresh tokens

- Rotate token family after successful renewal.
- Revoke previous family when replay is detected.
- Cover expiry and token-reuse scenarios.

Closes #184
```

Breaking-change example:

```text
feat(api)!: replace legacy user endpoint

- Add `/v2/me` for current-user details.
- Remove deprecated `/v1/users/current` handler.

BREAKING CHANGE: Clients must migrate to `/v2/me`.
```

## Usage

From target repository, invoke skill with a clear scope.

```text
Use professional-commit to audit current changes and prepare commit plan.
```

```text
Use professional-commit to create separate commits for current auth and docs changes.
```

```text
Use professional-commit to review staged changes and propose better commit message.
```

Expected approval gate:

```text
No files have been staged or committed by this skill. Approve this exact commit plan to proceed, or revise commit scope/message.
```

## Safety boundaries

- Explicit approval required before staging or committing.
- Existing staged changes are treated as user-owned. Skill never unstages them with `git reset`.
- Secret-like content stops commit planning until reviewed. Secret values are never printed.
- Required validation failure blocks normal approval. Commit despite failure needs explicit exception.
- Hooks remain enabled. Skill never uses `--no-verify` without explicit user request.
- Skill never pushes, amends, rewrites history, force-pushes, stashes, cleans, or changes Git config by default.
- Skill never commits broad paths through `git add .`, `git add -A`, or `git commit -a`.
- Partial-file commits require exact hunk review; unrelated hunks are not silently included.

## Install

Install for OpenCode, Claude Code, and Codex:

```sh
npx skills add AhmadIkbalDjaya/professional-commit -a opencode -a claude-code -a codex -y
```

Install only for OpenCode:

```sh
npx skills add AhmadIkbalDjaya/professional-commit -a opencode -y
```

## Skill contents

- `SKILL.md` — full audit, planning, approval, commit, and verification workflow.
- `README.md` — feature overview, message format, examples, usage, and safety limits.
