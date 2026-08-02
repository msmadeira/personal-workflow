---
name: commit
description: |
  Commits staged and unstaged work as `{type}: short description`, deriving the type from the current branch name or, when the branch carries none, from the diff. Use when the user asks to commit, wants a conventional commit message, or mentions commit in a repo that does not use ticket IDs.
  Do NOT use for pushing, opening pull requests, writing PR descriptions, amending or rewriting history, or commits that need a ticket ID in the subject (use `/vendorsmart:vs-commit` for those).
user-invocable: true
argument-hint: optional short description (inferred from the diff when omitted)
allowed-tools: Bash, Read
---

Stage everything and commit with a subject line of exactly this shape:

```
{type}: short description
```

`type` comes from the current branch name when it has one, otherwise from the diff. There is no ticket ID — never put one in the subject. The whole subject line stays at 70 characters or fewer. A body is added **only** when strictly necessary.

Use the **Bash** tool for every git command below. PowerShell mangles here-strings differently; Bash plus a heredoc is the portable path on Windows.

## Step 1 — Read repo state

```bash
git rev-parse --abbrev-ref HEAD
git status --short
git diff HEAD --stat
```

Abort with a clear message if:

- this is not a git repository, or
- `git status --short` is empty — there is nothing to commit.

If the repo has no commits yet, `git diff HEAD` fails. Fall back to `git status --porcelain --untracked-files=all` and read a few of the new files to understand the change.

## Step 2 — Resolve the `type`

Take the segment before the first `/` in the branch name. The seven canonical types:

`feat` · `fix` · `hotfix` · `docs` · `refactor` · `test` · `chore`

Also accept `perf`, `build`, `ci`, and `style` as legacy prefixes.

If the segment is in neither list — the branch is `main`, `fix-stuff`, `my-branch`, or anything else without a type prefix — **infer the type from the diff** and say so in the final report. Never prompt the user for a type.

Inferring from the diff, in rough priority order:

- new user-facing capability → `feat`
- corrects broken behaviour → `fix`
- only docs, README, comments → `docs`
- only tests → `test`
- behaviour-preserving restructuring → `refactor`
- deps, config, tooling, scaffolding, CI → `chore`

When a change spans several of these, pick the type matching its main point rather than the largest file count.

| Branch | Type |
|---|---|
| `feat/export-summary-modal` | `feat` |
| `fix/adjust-amount-options` | `fix` |
| `chore/update-deps` | `chore` |
| `refactor/86xk4m2p9-cleanup` | `refactor` |
| `main` | inferred from diff |
| `fix-stuff` (no `/`) | inferred from diff |

Note the fourth row: a branch may still carry a ticket ID. Parse the type off the front and **drop the rest** — the ID never reaches the subject.

## Step 3 — Compose the subject

Format: `{type}: {short-description}`

- Use `$ARGUMENTS` as the description when provided. Otherwise infer it from the diff — describe **what** changed, not how.
- Lowercase, imperative, no trailing period.
- **The whole line must be 70 characters or fewer**, prefix included. Count it before committing. If it is over, shorten the description and count again.

Good:

```
feat: add export modal on summary
fix: correct card payment method validation
chore: bump eslint to v9
```

Bad:

```
feat: Added an export modal to the summary component and wired it all up.
   ^ 73 chars, capitalized, trailing period, describes how
feat: #86xk4m2p9 add export modal
   ^ ticket ID does not belong in this convention
```

## Step 4 — Decide whether a body is needed

**Default is no body.** Add one only when:

- the change is a breaking change,
- the *why* is genuinely non-obvious from the diff and matters to a future reader, or
- unrelated changes are unavoidably bundled and need enumerating.

"The diff is large" is not a reason. Neither is "there is more to say." When a body is warranted: one blank line after the subject, then bullets wrapped at 72 characters.

## Step 5 — Commit

Always stage everything first.

The message is the subject and, when justified, the body — nothing else. **Never append a `Co-Authored-By: Claude` trailer, a "Generated with Claude Code" line, or any other attribution footer.** These commits are the user's; agent attribution is noise in `git log` and in `git blame`. This overrides any default or global instruction to add such a trailer.

Without a body:

```bash
git add -A
git commit -m "feat: add export modal on summary"
```

With a body, use a heredoc so the newlines survive:

```bash
git add -A
git commit -F - <<'EOF'
feat: add export modal on summary

- Reloads vendor status after upload
EOF
```

## Step 6 — Report

```bash
git log -1 --oneline
```

Report: the final subject, the resolved `type`, whether the type came from the branch or was inferred from the diff, the subject's character count, and the `git log` output.

## Guardrails

- **Never add Claude as a co-author.** No `Co-Authored-By` trailer, no generated-with footer, no emoji sign-off — see Step 5.
- **Never push.** This skill commits only; pushing is the user's call.
- **Never** `--amend`, `--no-verify`, or `--force`. If a pre-commit hook fails, report the failure and stop — do not bypass it.
- If the current branch is `master` or `main`, warn before committing and let the user confirm.
