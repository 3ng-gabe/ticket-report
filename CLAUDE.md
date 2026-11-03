# Ticket report job (cloud scheduled task)

When asked to "run the ticket report", do the following and nothing else.
Do not edit, commit, or push anything in this repository. Do not modify any Zoho tickets.

## 1. Gather tickets (Zoho Desk connector, org 871303623)
Departments:
- Engineering: 1056456000039476031
- IT Department: 1056456000094430464
- Security Department: 1056456000094438059

For each department, list every ticket whose statusType is not "Closed" (Open, On Hold, Escalated, etc.).
Page through results until none are left. For each ticket, read its conversation (threads + comments).

## 2. Analyze each ticket
- Sentiment: positive / neutral / frustrated / angry, based on the requester's own messages.
  Internal notes are context only.
- Where it stands: one sentence.
- Needed to close: the concrete next step(s).
- Waiting on: customer / RingLogix / vendor / unclear.
- Risk: low / medium / high. Flag anything idle 3+ days.

## 3. Post to Teams
Build an Adaptive Card (version 1.4) with:
- A header: "Ticket report: <day, time CT>" (convert the current time to America/Chicago)
- Per department: open count and sentiment mix
- Every high-risk or stale ticket: number (linked to its webUrl), subject, next step
- Medium/low tickets: one line each, number + subject + next step
Keep the card under 25 KB. If it would be larger, list only high and medium risk and say how many low-risk tickets were omitted.

Write the payload to /tmp/card.json in this exact envelope:
{"type":"message","attachments":[{"contentType":"application/vnd.microsoft.card.adaptive","content":<card>}]}

Post it, and print the HTTP status:
curl -sS -o /dev/null -w "HTTP %{http_code}\n" -X POST -H "Content-Type: application/json" -d @/tmp/card.json "$TEAMS_WEBHOOK_URL"

Success is HTTP 202. If TEAMS_WEBHOOK_URL is empty, or the post returns anything other than 202, say so clearly at the end of the run.
