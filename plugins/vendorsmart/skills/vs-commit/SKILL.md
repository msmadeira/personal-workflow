---
name: vs-commit
description: |
  Commits staged and unstaged work using the VendorSmart convention `{type}: #{ID} short description`, deriving the type and ticket ID from the current branch name. Use when the user asks to commit, wants a conventional commit message, mentions vs-commit, or asks for a commit following the branch/ticket pattern.
  Do NOT use for pushing, opening pull requests, writing PR descriptions, amending or rewriting history, or commits in repos that do not follow `<type>/<id>-<description>` branch naming.
user-invocable: true
argument-hint: optional short description (inferred from the diff when omitted)
allowed-tools: Bash, Read
---

Stage everything and commit with a subject line of exactly this shape:

```
{type}: #{ID} short description
```

`type` and `ID` come from the current branch name. The whole subject line stays at 70 characters or fewer. A body is added **only** when strictly necessary.

Use the **Bash** tool for every git command below. PowerShell mangles `#` and here-strings differently; Bash plus a heredoc is the portable path on Windows.

## Step 1 — Read repo state

```bash
git rev-parse --abbrev-ref HEAD
git status --short
git diff HEAD --stat
```

Abort with a clear message if:

- this is not a git repository, or
- `git status --short` is empty — there is nothing to commit.

## Step 2 — Parse the branch into `type` and `ID`

Branch format is `<type>/<id>-<description>` or `<type>/<id>/<description>`.

**`type`** — the segment before the first `/`. The seven canonical types, from the Web Git Flow doc:

`feat` · `fix` · `hotfix` · `docs` · `refactor` · `test` · `chore`

Also accept `perf`, `build`, `ci`, and `style` as legacy prefixes. They are not part of the convention and `vs-create-branch` will never produce them, but recognizing them keeps the ticket ID parseable on older branches.

If the segment is in neither list (e.g. the branch is `fix-stuff` or `my-branch`), infer the type from the diff instead and say so in the final report.

**`ID`** — take the segment after `type/` up to the first `/`, then test it against these patterns **in order**. The order matters.

1. **`NO-TICKET` sentinel** — the segment starts with `NO-TICKET`, `NOTICKET`, or `no-ticket` (any case) → `NOTICKET`
2. **Jira-style** — matches `^[A-Z]+-\d+`, e.g. `ABC-1234`. Test this **before** splitting on `-`, otherwise `ABC-1234` truncates to `AP`
3. **ClickUp** — take everything up to the first `-`; accept it only if it matches `^[a-z0-9]{8,10}$` **and contains at least one digit**, e.g. `86xk4m2p9`
4. **Otherwise** → `NOTICKET`

The digit requirement and the 8-character floor in rule 3 are load-bearing. Without them, ordinary descriptive branches get a word promoted to a ticket ID — `chore/update-deps` would commit as `chore: #update deps`, and `feat/user-profile-settings` as `feat: #user profile settings`.

Never prompt the user for a ticket ID — fall back to `NOTICKET` silently.

| Branch | Type | ID |
|---|---|---|
| `feat/86xk4m2p9-export-summary-modal` | `feat` | `86xk4m2p9` |
| `feat/86tq7wz13/invoices-list` | `feat` | `86tq7wz13` |
| `fix/86bn5r8v4/adjust-amount-options` | `fix` | `86bn5r8v4` |
| `fix/ABC-1234-card-payment-method` | `fix` | `ABC-1234` |
| `refactor/ABC-12-cleanup` | `refactor` | `ABC-12` |
| `test/NO-TICKET-fix-test` | `test` | `NOTICKET` |
| `chore/update-deps` | `chore` | `NOTICKET` |
| `feat/user-profile-settings` | `feat` | `NOTICKET` |
| `fix-stuff` (no valid type) | inferred from diff | `NOTICKET` |

## Step 3 — Compose the subject

Format: `{type}: #{ID} {short-description}`

- Use `$ARGUMENTS` as the description when provided. Otherwise infer it from `git diff HEAD` — describe **what** changed, not how.
- Lowercase, imperative, no trailing period.
- **The whole line must be 70 characters or fewer**, prefix included. Count it before committing. If it is over, shorten the description — never drop the `#{ID}`. Still over? Shorten again rather than committing a long subject.

Good:

```
feat: #86xk4m2p9 add export modal on summary
fix: #ABC-1234 correct card payment method validation
chore: #NOTICKET bump eslint to v9
```

Bad:

```
feat: #86xk4m2p9 Added an export modal to the summary component and wired it all up.
   ^ 84 chars, capitalized, trailing period, describes how
fix: adjust amount options
   ^ missing #{ID}
```

## Step 4 — Decide whether a body is needed

**Default is no body.** Add one only when:

- the change is a breaking change,
- the *why* is genuinely non-obvious from the diff and matters to a future reader, or
- unrelated changes are unavoidably bundled and need enumerating.

"The diff is large" is not a reason. Neither is "there is more to say." When a body is warranted: one blank line after the subject, then bullets wrapped at 72 characters.

## Step 5 — Commit

Always stage everything first.

Without a body:

```bash
git add -A
git commit -m "feat: #86xk4m2p9 add export modal on summary"
```

With a body, use a heredoc so the newlines survive:

```bash
git add -A
git commit -F - <<'EOF'
feat: #86xk4m2p9 add export modal on summary

- Reloads vendor status after upload
EOF
```

## Step 6 — Report

```bash
git log -1 --oneline
```

Report: the final subject, the resolved `type` and `ID`, the subject's character count, whether the ID was parsed from the branch or fell back to `NOTICKET`, and the `git log` output.

## Guardrails

- **Never push.** This skill commits only; pushing is the user's call.
- **Never** `--amend`, `--no-verify`, or `--force`. If a pre-commit hook fails, report the failure and stop — do not bypass it.
- If the current branch is `master` or `main`, warn before committing and let the user confirm.
