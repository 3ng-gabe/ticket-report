# ClickUp engineering report (cloud scheduled task)

When asked to "run the ClickUp engineering report", do the following and nothing else.
This job is READ-ONLY in ClickUp: never create, edit, comment on, or change the status of any task.
Do not edit, commit, or push anything in this repository.

## 1. Gather tasks (ClickUp connector)
Pull every open (not closed) task, including subtasks, from these lists:

| List | ID |
|---|---|
| Pre-Launch (Agent Studio) | 901418167982 |
| Launch Week | 901418168002 |
| Agent Studio | 901419052731 |
| Outbound Calls | 901419810006 |
| HIPAA compliance | 901415675932 |
| Acrobits v1 | 901415729049 |
| Dealer Module V1 | 901415675775 |
| Flowbots in RingOS: MVP | 901415655387 |
| Q3 Sprint Backlog | 901416333056 |
| Q4 Sprint Backlog | 901416333065 |
| RingOS backlog | 901416329636 |
| RingOS bugs | 901418368231 |
| RingOS feature requests | 901418558338 |
| Billing / QuickBooks | 901417943264 |
| RingOS critical services | 901418137479 |
| Data collector research | 901416329664 |
| Q4 Strategic Goals (Engineering) | 901421681771 |
| Q4 Strategic Goals (IT / Infra) | 901416877268 |
| Sprint 17 | 901418068407 |
| Sprint 18 | 901418551804 |
| Post-Launch (engineering items only) | 901418168014 |

Page through every result (keep requesting the next page until there are no more).

Skip:
- In Post-Launch, any marketing/GTM task (webinars, ads, outreach, videos, content, training sessions).
- Tasks tagged "not needed", or whose names start with "Not needed" or "Not relevant".
- Count tasks marked [Postpone]/[Postopone] but don't detail them.

## 2. Work out the report window
Runs are Tuesday and Thursday at noon America/Chicago.
- On Tuesday, "since last report" = since the previous Thursday 12:00 Central.
- On Thursday, "since last report" = since Tuesday 12:00 Central.
Use each task's date_updated to decide whether it changed in the window.

## 3. Analyze
For each task decide:
- **Changed since last report:** updated in the window; say what changed if visible (status, assignee, new comment).
- **Overdue:** due date is in the past.
- **Stale:** no update in 14+ days and not in a backlog/planned status.
- **Blocked:** status is "blocked", or the latest comments describe a blocker.
- **Needed to close:** for every changed, overdue, stale, blocked, urgent or high-priority task, read its description and latest comments and state the concrete next step(s) and who owns them. If it's unclear, say "next step unclear: needs owner input".

## 4. Post to Teams
Build an Adaptive Card (version 1.4), in this order:
1. Header: "ClickUp engineering report: <day, date, time CT>", plus totals (open, changed, overdue, stale, blocked, postponed).
2. **Changed since last report:** task name (linked to its url), list, status, what changed.
3. **Blocked:** task, owner, blocker, needed to close.
4. **Overdue:** task, owner, days overdue, needed to close.
5. **Stale (14+ days):** task, owner, days since last update.
6. **Urgent/high still open:** task, owner, status, needed to close.
A task appearing in an earlier section is not repeated in later sections.

Keep the card under 25 KB. If it would be larger, shorten the "needed to close" text first, then cut section 6 to the 10 oldest items and say how many were omitted.

Write the payload to /tmp/clickup_card.json in this exact envelope:
{"type":"message","attachments":[{"contentType":"application/vnd.microsoft.card.adaptive","content":<card>}]}

Post it and print the HTTP status:
curl -sS -o /dev/null -w "HTTP %{http_code}\n" -X POST -H "Content-Type: application/json" -d @/tmp/clickup_card.json "$TEAMS_WEBHOOK_URL"

Success is HTTP 202. If TEAMS_WEBHOOK_URL is empty or the post returns anything else, say so clearly at the end of the run.
