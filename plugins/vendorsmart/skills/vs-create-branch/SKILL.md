---
name: vs-create-branch
description: |
  Creates a git branch from a ClickUp task ID, following the VendorSmart Web Git Flow convention `{type}/{ID-or-NOTICKET}-{description}`. Infers the type from the task and builds a short kebab-cased description from its name. Use when the user wants to start work on a ClickUp task, asks to create or cut a branch for a ticket, mentions vs-create-branch, or gives a task ID and wants a branch for it.
  Do NOT use for committing (use vs-commit), pushing, opening pull requests, renaming or deleting branches, or creating branches in repos that do not follow the VendorSmart branch convention.
user-invocable: true
argument-hint: ClickUp task ID (e.g. 86agf2tw8), optionally followed by an explicit type
allowed-tools: Bash, Read, mcp__claude_ai_ClickUp__*
---

Create and switch to a branch named:

```
{type}/{ID-or-NOTICKET}-{description}
```

`type` is inferred from the ClickUp task, `ID` is the task ID, and `description` is a short kebab-cased summary of the task name.

Requires a connected ClickUp MCP server. If no ClickUp tool is available, say so and ask the user for the task's title and type rather than guessing.

## Step 1 — Resolve the input

`$ARGUMENTS` is a ClickUp task ID, optionally followed by an explicit type (`86agf2tw8 refactor`).

- An explicit type always wins over inference. Validate it against the table in Step 3.
- No ID given → ask for one. If the user says there is no ticket, use `NOTICKET` and ask for a short description.
- ClickUp IDs look like `86agf2tw8`; custom IDs like `DEV-1234` also work.

## Step 2 — Fetch the task

Call the ClickUp get-task tool with the ID and **no `include` parameter**.

The lean response already carries everything needed — `task_type`, `name`, `tags`, `status`, and the description. Do **not** pass `include: ["custom_fields"]`: it returns every dropdown option on the workspace (thousands of tokens) and adds nothing useful here.

If the task is not found, report the ID back and stop. Do not invent a description.

## Step 3 — Infer the type

The seven valid types, from the Web Git Flow doc:

| Type | Use for |
|---|---|
| `feat` | new feature for the user |
| `fix` | bug fix for the user |
| `hotfix` | bug hotfix for the user, directly in production |
| `docs` | changes to the documentation |
| `refactor` | refactoring code |
| `test` | adding missing tests, refactoring tests |
| `chore` | updating tasks, tools, etc. |

Resolve in this order; stop at the first match.

1. **Explicit type from `$ARGUMENTS`** — always wins.
2. **`task_type` field** — `Bug` → `fix`. Any other value (`BAU`, `Task`, …) is not a type signal on its own; fall through.
3. **Tags** — a tag of `bug` → `fix`; `tech-debt` → `refactor`; `documentation` → `docs`.
4. **Keywords in the task name, then the description:**

   | Signal | Type |
   |---|---|
   | fix, bug, broken, error, incorrect, not working, should be | `fix` |
   | refactor, cleanup, tech debt, restructure, migrate, rename | `refactor` |
   | test, coverage, e2e, unit test, spec | `test` |
   | document, readme, docs, guide | `docs` |
   | bump, upgrade, dependency, config, tooling, ci, pipeline | `chore` |
5. **Default** → `feat`.

**Never infer `hotfix`.** It means patching production directly, so it only applies when the user asks for it explicitly. If keywords suggest urgency (`urgent`, `production down`, `prod issue`), use `fix` and mention that `hotfix` is available if they want to patch production.

Always report which rule fired, so a wrong guess is easy to spot.

## Step 4 — Build the kebab description

Summarize the task **name** (not the description) into a short label.

**Rephrase, do not truncate.** Write the shortest imperative verb phrase that identifies the work, then kebab-case it. Deleting stopwords from the title left-to-right and cutting it off is the wrong approach — it keeps noise words and drops the point. `Vendor should be redirected to generate W9 when trying to manually uploading it` becomes `redirect-vendor-generate-w9`, not `vendor-redirected-generate-w9-trying`.

- Lead with the action as an imperative: `should be redirected` → `redirect`, `Edit …` → `edit`.
- Keep only what distinguishes this task. Drop circumstantial clauses (`when trying to…`, `manually`, `if possible`) and words a reader can infer (`list` in `waivers list` when `waivers` is already there).
- Output must match `^[a-z0-9]+(-[a-z0-9]+)*$` — lowercase alphanumerics separated by single hyphens. No `--`, no leading or trailing `-`.
- Target **3–5 words and 40 characters or fewer**. If it does not fit, cut the least distinguishing word, not the verb.

| Task name | Description |
|---|---|
| `Edit Management Company of a Subscription` | `edit-management-company` |
| `Vendor should be redirected to generate W9 when trying to manually uploading it` | `redirect-vendor-generate-w9` |
| `Include management company on waivers list` | `include-management-company-waivers` |
| `Adjust amount options` | `adjust-amount-options` |

Before moving on, check the result against the regex and the 40-character limit.

## Step 5 — Pick the base branch

Fixed branches per the Git Flow doc: `main` deploys to test, `release` deploys to production. Both are blocked for direct commits.

- `hotfix` → base off `release`, since the fix targets production. **Confirm with the user before proceeding.**
- Everything else → base off `main`.

If the repo has neither (e.g. it uses `master`), use its actual default branch and say which one you used.

Create from an up-to-date base without disturbing the working tree:

```bash
git fetch origin
git rev-parse --verify origin/main    # confirm the base exists
```

## Step 6 — Create the branch

```bash
git switch -c feat/86ag91vwj-edit-management-company origin/main
```

Before creating, check for problems and stop to ask rather than pushing through:

- **Branch already exists** (`git rev-parse --verify <name>`) → offer to switch to it instead of creating.
- **Uncommitted changes** (`git status --short` non-empty) → warn that they will follow you onto the new branch, and confirm.

## Step 7 — Report

```bash
git branch --show-current
```

Report the branch name, the resolved type and which rule inferred it, the base branch, and the task title and URL so the user can confirm it is the right ticket.

## Guardrails

- **Never push.** Creating the branch locally is the whole job.
- Never delete, rename, or force-move an existing branch.
- Never commit — that is `vs-commit`.
- If the task ID cannot be resolved, stop and report. Do not fall back to `NOTICKET` silently; `NOTICKET` is only for when the user says there is no ticket.
