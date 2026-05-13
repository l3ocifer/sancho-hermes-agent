---
name: hourly-checkin
description: Generate lightweight hourly check-ins for Leo by scanning calendar, open loops, inbox, and recent activity. Designed to produce a useful summary even when some data sources are empty.
trigger: Cron job scheduled hourly OR user request for hourly check-in
---

# Hourly Check-in for Leo

## Purpose

Generate lightweight hourly check-ins that:

- Summarize calendar, open loops, inbox, and recent activity.
- Keep Leo aware of anything Sancho should surface.
- Stay conversational and concise (max 2 short paragraphs).
- Include at most one direct question, and only if user input would unblock something.

## Failure-mode rule (read first — fixes the 2026-05-12 bug)

**Missing or empty data files are NORMAL, not a failure.** The whole point of an
hourly check-in is "what's new since last hour?", which is often "nothing".
A check-in that runs but reports no activity is a successful check-in.

Only emit the `⚠️ Hourly check-in failed — required tools unavailable` template when
**every single read path below errors out at the OS level** (e.g. graph PVC not
mounted, hermes runtime can't read any file). Specifically, `ENOENT` on
`pages/world/calendar-context.md` is **not** a failure — it just means today's
calendar sync hasn't populated that page yet (or there are no events). Report
"no events on file" for that section and continue.

The same rule applies to open-loops, inbox, and messaging tools: if a tool errors
on *its* source, that section gets a one-line "no data" note and the check-in
continues with whatever else is available.

## Workflow

### 1. Read each data source. Empty/missing → "no data", not failure.

For each source below, attempt the read. On `ENOENT` / "file not found" /
"tool unavailable", set that section's content to `_(no data this hour)_` and
move on. Never abort the whole check-in for a single missing source.

#### A. Calendar context — `/data/graphs/leo/pages/world/calendar-context.md`

Skim today's and tomorrow's events. If the page is empty / `_(no events on file)_` / has only frontmatter, your line is:

> No calendar items on file for the next 24h.

#### B. Open loops — `/data/graphs/leo/pages/world/open-loops.md`

Skim the `## Active` section. Surface items that are:

- Owned by Sancho (you), unresolved.
- P0/P1 priority regardless of owner.
- Stale > 7 days (mention "@owner — this has been waiting").

If the section is empty / bootstrap-only, your line is:

> No active open loops.

#### C. Inbox — `/data/graphs/sancho/pages/inbox/*.md`

List unread files (those without a `processed: true` frontmatter field). For each, show the file's
`task_id` (or filename if absent) and the first non-frontmatter line of the body. Cap at 5; if more,
say `… and N more in inbox`.

If the inbox is empty / missing, your line is:

> No unread inbox items.

#### D. Messaging — Himalaya / iMessage / Matrix

Per the agent's configured channels, check for unread Leo-directed messages with
P1+ urgency. If the underlying tool errors, note `(messaging tools currently unavailable)` and skip.

#### E. Recent agent activity — hermes session memory

Skim the last 60 minutes of your own session log for anything user-actionable that
hasn't been surfaced yet (e.g. a finished task you should report).

### 2. Compose the check-in

Format constraints:

- Maximum 2 short paragraphs OR 5 bullet points (whichever fits cleaner).
- At most one direct question, and only when input would unblock something.
- Conversational, lightweight; this isn't a status report.
- When EVERY section is empty: still send a 1-paragraph "all clear" check-in. **Do not skip the send** — Leo wants the heartbeat itself, not just the contents.

### 3. Output templates

#### With activity

```
Hourly check-in:

- {1-3 bullets covering the non-empty sections, most-important first}
- {optional: "Otherwise all clear"}

{optional one question}
```

#### Mostly empty

```
Hourly check-in: nothing urgent.

All sections clear (calendar, open loops, inbox, messaging). I'll keep watching.
```

#### True failure (extremely rare — graph PVC unmounted etc.)

Only if a tool failure prevented checking everything, including bootstrap pages
that should always exist:

```
⚠️ Hourly check-in could not run — substrate unavailable.

The leo-graph PVC is not mounted (or hermes runtime can't read /data/graphs/leo).
This is a real infrastructure issue, not just empty data. Investigation needed.
```

### 4. Deliver

Send via the channels configured in the cron job's `delivery` list (`bluebubbles` for the standard hourly-user-checkin).

## Tips

- Treat the inbox files as your highest-signal source. A file in
  `/data/graphs/sancho/pages/inbox/` exists because somebody (Leo or another agent)
  deliberately put it there. Process it.
- Be brief. Leo gets these every hour during waking hours; if you fill his attention every time, he'll mute you.
- Quiet hours are honored at the cron-task level (delivery side), not in the skill. You may still run; the message just gets deferred or dropped.
