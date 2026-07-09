# Oura daily import (required — not optional in this build)

Unlike the Strava build (where Strava is read live via a connector, and file-syncing to the
dashboard is an optional add-on), this Oura build has **no live connector at all** — Cowork's
connector directory doesn't have one. The *only* way Oura data reaches either the dashboard or
chat is this scheduled task, which calls Oura's REST API directly and writes one JSON file per
day. Set this up right after the dashboard is created; nothing shows an Oura number until it
has run at least once.

## Before creating the task: allow network access to Oura's API

Cowork's sandbox blocks outbound network calls made by *code Claude runs* (as opposed to
WebFetch/WebSearch or MCP tools, which aren't affected) unless the destination domain is
explicitly allowed. A scheduled task calling `api.ouraring.com` directly falls into the
blocked category by default — **this is almost certainly why an earlier attempt at this
failed with something like a blocked/forbidden network error.**

Fix it once, before creating the task:

1. Go to **Settings → Capabilities** (individual/Pro/Max accounts: `claude.ai/settings/capabilities`;
   Team/Enterprise: your org's admin settings → **Capabilities**).
2. Find the **Code execution** section's network / domain-allowlist control.
3. Either switch it to **"All domains"**, or keep it restrictive and add
   `api.ouraring.com` to the **"Additional allowed domains"** field.
4. **Start a new Cowork session/chat after saving this** — the setting applies to newly
   created sessions, not ones already running. If you change it mid-session, the change
   won't take effect until you open a fresh one.
5. If you added just `api.ouraring.com` and the task still gets blocked, try "All domains"
   instead — there have been intermittent reports of the single-domain allowlist not taking
   effect reliably. If neither works, that's a platform issue, not a setup mistake — try
   again later or ask in the Claude support channel.

You do **not** need to allowlist `cloud.ouraring.com` — that's only visited manually in your
own browser to generate the access token in the next step, not called by Cowork.

## Get a personal access token

1. Go to https://cloud.ouraring.com/personal-access-tokens and log in.
2. Click **Create New Personal Access Token**, name it something like "Health Tracker", and
   copy the token. Oura only shows it once — save it somewhere safe.

## Create the scheduled task

In Cowork, paste this prompt, replacing `YOUR_TOKEN_HERE` with the token you just copied.
**Name the task exactly `oura-daily-import`** when Cowork asks — the dashboard's "↻ Import
Oura" button and the "run my Oura import now" chat shortcut both look for that exact name.

```
Create a scheduled task named exactly "oura-daily-import" that runs every day at 11:00 PM.
The task should:

1. Call the Oura API endpoints for daily_activity and daily_readiness for today
   (auth header: "Bearer YOUR_TOKEN_HERE", base URL https://api.ouraring.com/v2/usercollection/)
2. Save the result as a JSON file at:
   <your HealthTracker folder>/oura/YYYY-MM-DD.json
   with this shape:
   {
     "date": "YYYY-MM-DD",
     "active_calories": <number from daily_activity.active_calories>,
     "total_calories": <number from daily_activity.total_calories>,
     "steps": <number>,
     "high_activity_minutes": <number from daily_activity.high_activity_time / 60>,
     "readiness_score": <number from daily_readiness.score>
   }

Also run the task once right now so I have today's file.
```

Approve the task when Cowork asks. Once it's run at least once, both the dashboard and chat
("what's my net today?") will pick up the numbers.

## Running it on demand

- **From the dashboard:** click "↻ Import Oura" (next to "Exercise today"). It triggers the
  `oura-daily-import` task and polls for the new file for up to ~2 minutes.
- **From chat:** say "run my Oura import now" — Claude will trigger the same task, or pull the
  data directly if it can't invoke the scheduled task from chat.

## Troubleshooting

- **Task fails with a network/permission error calling api.ouraring.com:** you're missing the
  Settings → Capabilities network-access step above, or created the task before a fresh session
  picked up the setting change.
- **Dashboard shows no Oura data even though the task ran:** open the 🔧 Load diagnostic panel
  at the bottom of the dashboard — it shows exactly which Oura file paths it looked for and
  what it got back. Usually a folder-path mismatch (confirm the task is writing into the same
  `HealthTracker/oura/` folder the dashboard's Filesystem connector is pointed at).
- **"↻ Import Oura" button errors immediately:** almost always means the scheduled task isn't
  named exactly `oura-daily-import`. Rename it (or recreate it) to match.
