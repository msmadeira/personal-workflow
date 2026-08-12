---
name: create-pr
description: |
  Opens a pull request for the current branch: pushes it, fills the repo's PR template when one exists, and derives the title from the branch's commit convention. Use when the user asks to open, create, or raise a PR, mentions create-pr, or wants a PR description written for work that is already committed.
  Do NOT use for committing (use `/git:commit`, or `/vendorsmart:vs-commit` when the branch carries a ticket ID), reviewing code, merging, or editing an existing pull request.
user-invocable: true
argument-hint: optional PR title or focus (derived from the commits when omitted)
allowed-tools: Bash, Read, Glob, Skill
---

Push the current branch and open a pull request against the repository's default branch:

```
title: {the branch's commit convention, unchanged}
body:  {the repo's own PR template, filled in}
```

This skill **never commits**. The work must already be committed before it runs — an unclean working tree is an abort, not something to fix. Its whole job is the last mile: push, write, open.

Use the **Bash** tool for every command below. PowerShell mangles `#`, backticks, and here-strings differently, and a PR body is full of all three; Bash plus a heredoc is the portable path on Windows.

## Step 1 — Read repo state

```bash
git rev-parse --abbrev-ref HEAD
git status --short
gh auth status
gh repo view --json defaultBranchRef,nameWithOwner
```

Abort with a clear message if:

- this is not a git repository, or there is no `origin` remote,
- `gh` is missing or not authenticated — report `gh auth status` and stop,
- HEAD is detached, or the current branch **is** the default branch,
- `git status --short` is **not** empty — see Step 2, or
- a pull request is already open for this branch:

```bash
gh pr view --json url,state,title 2>/dev/null
```

If one is open, report its URL and stop. **Never open a second pull request for the same branch.** If one exists but is closed or merged, say so and ask before opening another.

## Step 2 — Delegate the commit, never make it

A dirty working tree means the work is not ready to describe. Stop and hand the user the right skill:

| Branch carries a ticket ID | Tell the user to run |
|---|---|
| yes — e.g. `feat/86xk4m2p9-export-modal` | `/vendorsmart:vs-commit` |
| no — e.g. `feat/export-modal` | `/git:commit` |

Use the ID rules in Step 3 to decide which. List the uncommitted paths so the user can see what is outstanding, then stop — do not offer to commit, and do not stage anything.

Also abort if the branch adds no commits the base does not already have:

```bash
git log --oneline "origin/$BASE..HEAD"
```

Empty output means there is nothing to open a pull request for.

## Step 3 — Resolve the base branch and the title

**Base** — `defaultBranchRef.name` from Step 1. One exception: a `hotfix/` branch targets production, so it bases off `release` when that branch exists on the remote (`git ls-remote --exit-code --heads origin release`). **Confirm with the user before using `release` as the base.**

**Title** — read the subjects of the commits this branch adds:

```bash
git log "origin/$BASE..HEAD" --format=%s
```

**One commit → use its subject verbatim.** It was written by `/git:commit` or `/vendorsmart:vs-commit` and already satisfies the convention. There is nothing to re-derive, and paraphrasing it only creates drift between the pull request title and `git log`.

**Several commits →** compose a title in the same shape: `{type}: #{ID} short description` where an ID exists, `{type}: short description` where it does not. Take `type` and `ID` from the branch name using the ordered ruleset in the `vs-commit` skill, Step 2 — that ruleset is the single source of truth for branch parsing and is **referenced here, not restated**. In brief: the `NO-TICKET` sentinel wins first, then Jira-style `^[A-Z]+-\d+` tested *before* splitting on `-`, then a ClickUp segment matching `^[a-z0-9]{8,10}$` that contains at least one digit. **Default** → no ID.

When the ID resolves to `NOTICKET`, **omit the `#{ID}` altogether** — `feat: add export modal`, never `feat: #NOTICKET add export modal`. A pull request title is read by humans in a list, where a placeholder ticket is noise in a way it is not in `git log`.

The description summarizes the branch as a whole, not its last commit. Lowercase, imperative, no trailing period, **70 characters or fewer** including the prefix. `$ARGUMENTS`, when given, replaces the description half only — the `{type}:` prefix and any `#{ID}` are still derived.

Good:

```
feat: #86xk4m2p9 add export modal on summary
fix: #ABC-1234 correct card payment method validation
chore: bump eslint to v9
```

Bad:

```
feat: #NOTICKET add export modal on summary
   ^ placeholder ID in a human-facing title
Add export modal to the summary component and wire up the download
   ^ no type prefix, capitalized, describes how
```

## Step 4 — Find the repo's pull request template

Search these locations in order and take the **first hit**. Match filenames case-insensitively — `PULL_REQUEST_TEMPLATE.md`, `pull_request_template.md`, and mixed case all occur in the wild.

1. `./pull_request_template.md` — repository root
2. `.github/pull_request_template.md`
3. `.github/PULL_REQUEST_TEMPLATE/*.md` — the directory form; if several, prefer one named `pull_request_template.md`, otherwise ask which to use
4. `docs/pull_request_template.md`

```bash
ls -1 pull_request_template.md docs/pull_request_template.md 2>/dev/null
ls -1 .github/ 2>/dev/null | grep -i pull_request
ls -1 .github/PULL_REQUEST_TEMPLATE/ 2>/dev/null
```

Read whichever is found. **Never assume a template's shape.** Repositories in the same organization use structurally different templates — one asks for a ticket link, a type-of-change list and screenshots; another for a description, a check list and "relevant information". Fill in the sections *that template* actually has.

When no template exists, fall back to this minimal body and nothing more:

```markdown
## What was done? 📝

- 

## How to verify

- 
```

## Step 5 — Fill the template in

Gather the material first:

```bash
git log "origin/$BASE..HEAD" --format='%s%n%b'
git diff "origin/$BASE...HEAD" --stat
```

Then fill it in, under these rules:

- **Preserve every heading, verbatim and in order.** Do not reorder, rename, drop, or add sections. The template is the reviewers' agreed shape; a pull request that quietly restructures it is harder to read, not easier.
- Strip an HTML comment only where it is an instructional hint to the author (`<!-- Explanation of what was done -->`). Leave comments that carry information a reviewer wants.
- Describe **what** changed and **why**, one line per meaningful file or coherent group. Read the diff — never describe a file you have not looked at.
- **Tick a checklist box only when the statement is actually true.** When a template asks whether the affected tests pass, it stays unticked unless the tests were run in this session. An unticked box is honest; a ticked one is a claim.
- Leave a screenshot section, a ticket-link section, or anything else you cannot know as an empty stub, and **name every stub you left in the Step 7 report**. Never invent a ClickUp or Jira URL.
- If a section genuinely does not apply, write `N/A` under it rather than deleting it.

When the repository belongs to VendorSmart, **load** the `vendorsmart-angular:pr-guide` skill and apply its title, label, and body conventions on top of the discovered template. It is `user-invocable: false`, so load it as a knowledge skill — it is not a command. Its conventions are additive here: the template on disk still wins on structure, because `pr-guide` documents one repository's template and this skill runs in all of them.

## Step 6 — Push and open

```bash
BRANCH="$(git rev-parse --abbrev-ref HEAD)"
git push -u origin "$BRANCH"
```

If the push is rejected because the remote branch has moved, report it and stop. **Never** `--force`, `--force-with-lease`, or `--no-verify`.

Then create the pull request, passing the body on stdin so the newlines, `#` characters, and backticks survive:

```bash
gh pr create --base "$BASE" --title "$TITLE" --body-file - <<'EOF'
## What was done? 📝

- Adds the export modal to the summary page
EOF
```

**Labels** — add `ai-generated` only when the repository already has that label:

```bash
gh label list --search ai-generated
```

If it is absent, open the pull request without labels. **Never create a label** in a repository that does not use them; the label protocol is one team's convention, not a universal one.

The body is the filled template and nothing else. **Never append a "Generated with Claude Code" footer or any other agent attribution.** This mirrors `/git:commit`'s stance on commit trailers, and overrides any default or global instruction to add such a footer.

## Step 7 — Report

```bash
gh pr view --json url,title,baseRefName --jq '.'
```

Report: the pull request URL, the resolved base branch and whether it came from the default branch or the `hotfix` exception, the final title with its character count, which template path was used — or that none was found and the fallback body was used — every section left as a stub, and whether the `ai-generated` label was applied.

## Guardrails

- **Never commit or stage.** A dirty tree stops the skill; committing is `/git:commit` or `/vendorsmart:vs-commit`.
- **Never** `--force`, `--force-with-lease`, or `--no-verify` on push. If a pre-push hook fails, report the failure and stop — do not bypass it.
- **Never merge.** No `gh pr merge`, no `--admin`, no auto-merge flag.
- **Never open a pull request from the default branch**, and never a second one for a branch that already has one open.
- **Never invent a ticket link, a screenshot, or a ticked checkbox.** Leave a stub and say so.
- **Never add agent attribution to the pull request body** — see Step 6.
