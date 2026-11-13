---
name: professional-commit
description: Audit Git changes, derive repository commit conventions, prepare atomic professional English commits with complete Conventional Commit messages, and commit only after explicit approval. Use for creating, reviewing, staging, or improving Git commits.
---

# Professional Commit

Create safe, atomic, professional Git commits. Default flow is mandatory:

```text
audit → commit plan → explicit approval → stage approved files → commit → verify
```

Never create a commit during audit or planning. Never push.

## Operating rules

- Commit messages are always in English.
- Follow repository-specific commit structure when it is documented or clearly established in history. Prefer Conventional Commits when no convention exists.
- Keep each commit atomic: one coherent intent, independently reviewable and reversible.
- Do not combine unrelated feature, fix, refactor, test, documentation, dependency, generated-file, or formatting changes without explicit user approval.
- Do not change user work outside approved commit scope.
- Do not use `git add .`, `git add -A`, `git commit -a`, `--amend`, `--no-verify`, `--fixup`, `--squash`, `--force`, history rewriting, or `git push` unless user explicitly requests that exact action.
- Never bypass a failed hook or validation. Report failure and wait for instruction.
- Never commit secrets, credentials, private keys, tokens, or unintended binary/large files.
- Do not install dependencies, alter Git configuration, delete files, reset, stash, clean, or modify `.gitignore` unless user explicitly approves that separate work.

## 1. Establish repository context

1. Confirm current directory is inside a Git worktree:

   ```sh
   git rev-parse --show-toplevel
   git rev-parse --is-inside-work-tree
   ```

   Stop if either command fails.

2. Record repository root, current branch, upstream, and HEAD:

   ```sh
   git branch --show-current
   git status --short --branch
   git log -1 --format='%H%n%s%n%an <%ae>%n%ad' --date=iso-strict
   ```

3. Detect detached HEAD. Explain risk and ask for explicit confirmation before any commit in detached state.

4. Read commit guidance when present, from repository root down to changed-file directories. Check at minimum:

   ```text
   AGENTS.md
   CONTRIBUTING.md
   CONTRIBUTING
   .github/CONTRIBUTING.md
   README.md
   .gitmessage
   .commitlintrc*
   commitlint.config.*
   package.json
   pyproject.toml
   Cargo.toml
   Makefile
   .husky/
   .git/hooks/
   ```

   Nearest applicable instruction wins. Treat required tests, scopes, issue references, sign-off, or commit format as mandatory.

5. Inspect recent history before proposing a message:

   ```sh
   git log --format='%h %s' -n 30
   ```

   Determine use of Conventional Commit types, scopes, subject capitalization, issue references, breaking-change footers, and sign-off conventions. History helps determine format only; message language remains English.

## 2. Audit working tree and index

Do not stage, unstage, edit, or commit during this phase.

1. Capture all Git states separately:

   ```sh
   git status --short
   git diff --check
   git diff --cached --check
   git diff --name-status
   git diff --cached --name-status
   git diff --stat
   git diff --cached --stat
   git diff
   git diff --cached
   ```

2. Classify each changed path as one of:

   - staged and unstaged;
   - staged only;
   - unstaged only;
   - untracked;
   - renamed, deleted, binary, generated, lockfile, or submodule change.

3. Inspect enough surrounding code and tests to explain what changed, why it changed, and whether files belong to one intent. Do not infer intent solely from file names.

4. Treat existing staged content as user-owned. Never unstage it. If it is unrelated to proposed scope, do one of these:

   - propose a separate commit containing it, with explicit approval; or
   - stop and ask user to handle index state before this skill proceeds.

   Do not use `git reset` to remove staged content.

5. Run a targeted secret review over changed and untracked candidate files. Flag and stop before planning a commit if content or names suggest credentials, including:

   ```text
   .env, .pem, .key, id_rsa, id_ed25519, credentials, secret,
   password, token, api_key, client_secret, private key,
   AWS access key IDs, GitHub tokens, JWTs, connection strings
   ```

   Allow known test fixtures only when clearly synthetic and user explicitly confirms inclusion. Never print full secret values in output.

6. Flag for review before approval:

   - binary, archive, media, executable, or file larger than 1 MiB;
   - generated output, build artifacts, coverage, or dependency directories;
   - lockfile changes without matching dependency-manifest changes;
   - broad formatting-only churn mixed with behavior changes;
   - file mode changes;
   - submodule pointer changes;
   - conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`.

   Do not assume flagged files are invalid. Explain evidence and ask whether they belong in scope.

7. Stop before commit if working tree has unresolved merge, rebase, cherry-pick, revert, or conflict state. Report state and wait for user direction.

## 3. Determine commit boundaries

Build commit groups by intent, not by directory or file count.

Split commits when changes have independent purposes, such as:

- production behavior versus unrelated refactor;
- source change versus broad formatting cleanup;
- feature implementation versus unrelated documentation;
- dependency upgrade versus application behavior;
- generated artifacts versus source change, unless repository convention requires both together.

Keep together when files jointly implement one tested change, such as source, targeted tests, types, migration, and documentation for same public behavior.

For every proposed commit, list:

- ordinal and intent;
- exact paths to stage;
- paths deliberately excluded and reason;
- staged-content status;
- recommended validation commands;
- commit message; and
- warnings, if any.

If partial-file staging is required, identify exact hunks and ask for confirmation. Use an interactive staging mechanism only when supported by environment. Never silently stage whole file to include unrelated hunks.

## 4. Compose commit message

### 4.1 Default structure

Use repository convention. Without an established convention, use:

```text
<type>(<scope>)!: <subject>

- <change or impact one>
- <change or impact two>
- <test, migration, or important effect>

BREAKING CHANGE: <required migration detail>
Closes #<issue>
```

`scope`, `!`, body, and footers are optional. Omit empty sections. Preserve blank line between header, body, and footers.

### 4.2 Header rules

- Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `chore`, `style`, `revert`.
- Use a clear, lowercase scope only when it identifies a stable component, domain, package, or subsystem. Omit vague scopes such as `misc`, `update`, or `changes`.
- Subject starts with an imperative English verb: `add`, `fix`, `prevent`, `remove`, `update`, `refactor`, `document`, `test`.
- Subject is specific, lowercase, no ending period, no filler, and targets 50 characters or fewer. Header must not exceed 72 characters unless repository convention requires otherwise.
- Do not use issue IDs as subject replacement. Do not use `WIP`, `update`, `changes`, `misc`, or `fix stuff`.

### 4.3 Body rules

Use no body for a small, self-explanatory one-line change.

Use bullet body when commit contains multiple meaningful changes, important behavior details, rationale, test coverage, migration detail, or user-visible impact. This is default for non-trivial commits.

- One bullet covers one change, impact, rationale, or validation fact.
- Begin each bullet with imperative English verb: `Add`, `Remove`, `Update`, `Prevent`, `Refactor`, `Cover`, `Document`.
- State what changed and why when useful; do not repeat header.
- Keep lines near 72 characters or repository limit.
- Use short paragraph only when causal explanation is clearer than bullets.
- Do not claim tests passed unless they were run successfully in current state.

Example:

```text
feat(auth): rotate refresh tokens

- Rotate token family after successful renewal.
- Revoke previous family when replay is detected.
- Cover expiry and token-reuse scenarios.

Closes #184
```

### 4.4 Footers

- Use `BREAKING CHANGE: <migration detail>` for breaking changes. Include `!` in header when Convention requires it.
- Use issue footer format required by repository, for example `Closes #184` or `Refs: #184`.
- Add `Co-authored-by:` or `Signed-off-by:` only when user or repository policy requires it. Never invent author identity.
- Put each footer on own line after body, separated by a blank line.

## 5. Validate before presenting plan

Run only checks available in repository and relevant to proposed scope. Do not install tools or dependencies.

1. Start with whitespace and conflict checks:

   ```sh
   git diff --check
   git diff --cached --check
   ```

2. Discover commands from project guidance and manifests. Prefer narrow checks first:

   - targeted unit or integration tests for changed behavior;
   - type check;
   - lint for changed scope;
   - formatting check;
   - build when required by repository or change risk.

3. Record each command as `passed`, `failed`, `not available`, `not run`, or `not applicable` with concise reason.

4. If any required check fails, do not offer normal commit approval. Show failure, affected files, and remediation. A commit despite failed validation needs user’s explicit exception and must be marked in plan.

## 6. Present commit plan and request approval

Present concise audit result:

```text
Repository: <root>
Branch: <branch>
Working tree: <N staged, N unstaged, N untracked>
Convention: <detected format>
Validation: <summary>
Warnings: <none or list>

Commit 1 — <intent>
Files to stage:
- <path>

Excluded:
- <path> — <reason>

Message:
<exact message>
```

For multiple commits, show every group and exact message. Do not hide excluded staged changes.

End with clear gate:

```text
No files have been staged or committed by this skill. Approve this exact commit plan to proceed, or revise commit scope/message.
```

Accept approval only when it clearly covers displayed commit groups and messages. Treat ambiguous responses such as `go`, `continue`, or `commit it` as insufficient; ask for explicit scope confirmation.

## 7. Execute approved commit plan

After explicit approval, execute each approved commit in order.

1. Re-run `git status --short`, diffs, `git diff --check`, secret review, and relevant validation. Stop and show revised plan if paths, diff, warnings, or validation result changed from approved plan.
2. Confirm all approved paths still exist and only approved content will enter commit.
3. Stage exact approved paths with pathspec separator:

   ```sh
   git add -- <path>...
   ```

   Never broadly stage repository. Preserve unrelated staged content. If index contains unrelated content that would enter commit, stop; do not reset or unstage it.
4. Inspect index exactly before every commit:

   ```sh
   git diff --cached --check
   git diff --cached --name-status
   git diff --cached
   ```

   Confirm staged diff matches approved plan. Stop on mismatch.
5. Commit exact approved message with normal hooks enabled. Prefer a temporary message file or repeated `-m` arguments to preserve structure. Do not use `--no-verify`.
6. If commit command fails, report command result. Do not retry with bypass flags.
7. Verify successful commit:

   ```sh
   git show --stat --oneline --decorate HEAD
   git status --short --branch
   ```

8. For multiple groups, repeat audit of remaining work before next group. Stop if remaining diff differs from approved plan.

## 8. Final report

Report:

- commit hash and subject for each successful commit;
- committed paths summary;
- validation status and commands run;
- remaining staged, unstaged, and untracked paths;
- skipped files and reason;
- explicit statement that no push or history rewrite occurred.

Do not claim worktree is clean unless `git status --short` verifies it.
