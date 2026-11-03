---
name: progress-checklist
description: |
  Tracks a multi-step task with a checklist at the end of every reply until the work is done. The list comes from a ticket's acceptance criteria, a description of work to break down, or a list the user gives. It ticks items as they land and keeps open questions on the list until they're decided. Use when the user asks to track progress, keep a checklist, break something down into steps, follow ACs across several screens or files, or work through something "one place at a time" / "separately".
  Do NOT use for a single-step change, for a plan the user hasn't approved yet (plan first, then track), or to write a tracker file. The checklist only lives in the replies.
user-invocable: true
argument-hint: ticket ID/URL, a description of the work to break down, or a list of items
allowed-tools: Read, Grep, Glob, Bash
---

Keep a running checklist of a multi-step task and **end every reply with it** until every item is ticked or the user stops the tracking.

The checklist lives **only in the replies**. Never write it to a file (no `notes/*.md`, no `TODO.md`, no scratch tracker). Changing a file in step with the work just gives the user one more thing to look after. The latest reply is the source of truth.

## Step 1: Gather the items

Work out what `$ARGUMENTS` is, then seed the list from it:

| `$ARGUMENTS` | Seed from |
|---|---|
| A ticket ID or URL (`86xk4m2p9`, `DEV-1234`, a ClickUp/Jira link) | Fetch it with the tracker's MCP tool and read its acceptance criteria and technical notes. If no tool can reach it, say so and ask for the ACs instead of guessing. |
| A description of the work ("migrate the three list pages to SelectFilter") | Break it down into concrete steps. Check the codebase for the places it touches, and state the breakdown so the user can correct it before work starts. |
| A list (bullets, numbered lines, comma-separated) | Take the items as given, one checklist item each, in the user's order. Don't reword, merge or split them. Add only *All* items and open questions alongside. |
| Empty | Use the task already under way in the conversation, from its ACs, plan or the user's last request. If nothing in the conversation is multi-step, ask what to track. |

Then shape the items:

1. **Places that change.** Turn each screen, file or step into its own item. Find the real paths with Grep or Glob before listing them, and never list a file you haven't located.
2. **Shared criteria.** An AC that applies everywhere ("states unchanged on every screen", "not covered by toasts") becomes one item under *All*, not one copy per place.
3. **Open questions.** An ambiguity spotted while gathering (a button the ACs don't mention, a row that may be out of scope) becomes an `Open question:` item.

Keep each item to one line: the place, then what changes there. Detail such as IDs, conditions and handlers belongs in the reply's body when you work on that item, not in the checklist.

## Step 2: Show it

Put a header first, with a link to the ticket when there is one, then the list:

```markdown
**Tracking: [Move header form actions to Footer Actions](https://app.clickup.com/t/17ty0uzw9ya)**

- [x] Create management company: Save management company
- [ ] Create event: Create event
- [ ] Add bundle: Save as draft + Activate bundle (Preview, Delete stay in the header)
- [ ] All screens: disabled/loading states and submit behavior unchanged
- [ ] Open question: RFP "Go to vendor sourcing" isn't in the ACs, so it's undecided whether it moves or stays
```

End by asking which item to take next, unless the user already said.

## Step 3: Keep it current, every reply

Each reply about this task ends with the full checklist, updated:

- **Tick** an item only once it's actually done, meaning the change is made and verified as far as you can verify it (tests, lint, a checked diff). A ticked box is a claim.
- **Manual checks** the user has to do in a browser or device stay unticked until the user confirms them. Mark them `(check in the browser)` so it's clear the user owns them.
- **Resolve open questions** by rewording them as the decision (`RFP "Go to vendor sourcing": stays in the header (decided)`) and ticking them, or by turning them into a new work item.
- **Add** items when new work turns up, and **drop** items the user rules out, saying so in one line.
- Keep ticked items on the list so progress stays visible. Don't prune them.

Never change items silently. When the list changes for a reason other than finished work, say why in a sentence above it.

## Step 4: Finish

When every item is ticked, show the final list once with a one-line "all done" and stop adding it to later replies. If the user says to stop tracking, stop right away.

## Guardrails

- **No tracker files.** The checklist is never written to disk.
- **Every reply.** A reply about the task without the checklist breaks the tracking, and that includes short answers and side questions about the same task.
- **Honest ticks.** Don't tick unverified work or manual checks the user hasn't confirmed.
- **One checklist per task.** Don't start a second list for the same work. Extend the existing one.
