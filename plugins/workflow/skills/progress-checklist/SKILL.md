---
name: progress-checklist
description: |
  Tracks a multi-step task with a checklist at the end of every reply until the work is done. It builds the list from the ticket's acceptance criteria and the places that change, ticks items as they land, and keeps open questions on the list until they're decided. Use when the user asks to track progress, keep a checklist, follow ACs across several screens or files, or work through a ticket "one place at a time" / "separately".
  Do NOT use for a single-step change, for a plan the user hasn't approved yet (plan first, then track), or to write a tracker file. The checklist only lives in the replies.
user-invocable: true
argument-hint: optional ticket ID or URL to seed the checklist from its acceptance criteria
allowed-tools: Read, Grep, Glob, Bash
---

Keep a running checklist of a multi-step task and **end every reply with it** until every item is ticked or the user stops the tracking.

The checklist lives **only in the replies**. Never write it to a file (no `notes/*.md`, no `TODO.md`, no scratch tracker). Changing a file in step with the work just gives the user one more thing to look after. The latest reply is the source of truth.

## Step 1: Gather the items

Seed the list from what the work actually is:

1. **Acceptance criteria.** When `` is a ticket ID or URL, fetch it with the tracker's MCP tool (ClickUp, Jira, …) and read the ACs. Otherwise use the ACs from the conversation.
2. **Places that change.** Turn each screen, file or step from the ACs or technical notes into its own item. Find the real paths with Grep or Glob before listing them, and never list a file you haven't located.
3. **Shared criteria.** An AC that applies everywhere ("states unchanged on every screen", "not covered by toasts") becomes one item under *All*, not one copy per place.
4. **Open questions.** An ambiguity spotted while gathering (a button the ACs don't mention, a row that may be out of scope) becomes an `Open question:` item.

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
