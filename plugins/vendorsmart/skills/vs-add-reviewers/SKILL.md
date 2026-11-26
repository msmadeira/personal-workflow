---
name: vs-add-reviewers
description: |
  Requests the usual VendorSmart reviewers (lucasjardi, rafastrack123, guih2127, MarcioSchlichting) on a pull request — by default the one just opened for the current branch. Skips the PR author and anyone already requested or who already reviewed. Use when the user asks to add reviewers, request review, mentions vs-add-reviewers, or has just opened a VendorSmart PR with `/git:create-pr` and wants the team on it.
  Do NOT use for opening the pull request (use `/git:create-pr`), removing reviewers, approving, or merging.
user-invocable: true
argument-hint: optional PR number or URL (defaults to the current branch's open PR)
allowed-tools: Bash
---

Request review from the team's usual reviewers on one pull request:

```
lucasjardi · rafastrack123 · guih2127 · MarcioSchlichting
```

Use the **Bash** tool for every command below.

## Step 1 — Resolve the pull request

`$ARGUMENTS` is an optional PR number or URL.

- **Given** → use it as-is.
- **Omitted** → use the open pull request for the current branch.

```bash
gh pr view $ARGUMENTS --json number,url,state,author,reviewRequests,reviews \
  --jq '{number, url, state, author: .author.login,
         requested: [.reviewRequests[] | .login // empty],
         reviewed: ([.reviews[].author.login] | unique)}'
gh repo view --json nameWithOwner --jq .nameWithOwner
```

Abort with a clear message if:

- no pull request is found — tell the user to open one first with `/git:create-pr`, or
- `state` is not `OPEN` — report the state and stop.

## Step 2 — Work out who to request

Start from the four logins above, then drop:

- the PR **author** — GitHub rejects a review request to the author, and the whole call fails with it,
- anyone already in **requested**,
- anyone already in **reviewed** — re-requesting would reset their review to pending.

If nobody is left, report that everyone is already on the pull request and stop. Do not call the API.

## Step 3 — Request the reviews

Use the REST endpoint, one `reviewers[]` field per login. Do not use `gh pr edit --add-reviewer`: it goes through GraphQL and fails on repositories that still reference classic Projects.

```bash
gh api "repos/$REPO/pulls/$NUMBER/requested_reviewers" -X POST \
  -f 'reviewers[]=lucasjardi' -f 'reviewers[]=rafastrack123' \
  --jq '[.requested_reviewers[].login]'
```

A `422` naming a login means that user is not a collaborator on the repository. Drop that login, retry once with the rest, and report the one that failed.

## Step 4 — Report

One or two lines:

1. who was requested, and the pull request URL,
2. who was skipped and why (author, already requested, already reviewed, not a collaborator) — omit the line when nobody was skipped.

## Guardrails

- **Never remove a reviewer** or touch anything else on the pull request — title, body, labels, assignees.
- **Never approve, comment on, or merge** the pull request.
- **Never re-request** someone who already reviewed.
- **Never request review on a closed or merged** pull request.
