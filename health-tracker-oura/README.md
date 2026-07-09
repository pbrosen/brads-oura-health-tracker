# Health Tracker — Oura build

A personal daily **health tracker** for Claude Cowork — log your food, exercise, and
digestive (gut) symptoms by chatting with Claude, with a visual dashboard, and your
**Oura ring** feeding in daily activity and readiness. Everything is stored as **plain
Markdown (and one JSON file per day) in a folder on your own computer**. Nothing is
uploaded anywhere except the direct calls to Oura's own API to pull your data.

It does four things:

1. **Food + calories.** Tell Claude what you ate ("turkey sandwich and chips for lunch")
   and it logs it with estimated calories and macros, learning your common foods over time.
2. **Exercise + Oura.** Log manual workouts (weights, yoga) by chat or in the dashboard.
   A nightly scheduled task pulls your Oura ring's active calories, steps, high-activity
   minutes, and readiness score into your burn and net-calorie numbers.
3. **Digestion tracking.** Log gut episodes (diarrhea, loose stool, constipation,
   gas/bloating) with severity and Bristol score, then use the dashboard's
   **"Find food correlations"** button to see which foods tend to precede your symptoms.
4. **On-demand sync.** Click "↻ Import Oura" in the dashboard (or ask Claude to "run my
   Oura import now") to pull a fresh number any time, not just at the nightly sync.

About 30–45 minutes start to finish, most of it waiting on installers.

---

## What you'll need before starting

- **Claude desktop app** (macOS or Windows). If you don't have it:
  [install Claude Desktop](https://support.claude.com/en/articles/10065433-install-claude-desktop).
- **A paid Claude subscription** — Pro, Max, Team, or Enterprise. Cowork mode (which the
  tracker runs inside) isn't on the free tier.
- **An Oura account** with an active ring.
- **About 10 GB of free disk space** — Cowork's sandbox needs room.

You do **not** need Claude in Chrome — everything happens inside the desktop app.

---

## Setup — do these in order

No terminal, no Node, no config files. The dashboard saves to your computer through a
one-click **Filesystem** extension (it bundles everything it needs).

### 1. Enable Cowork mode

Open Claude desktop → click the **Cowork** tab at the top (next to "Chat"). If you've never
used it before, follow the brief onboarding.
([Get started with Claude Cowork](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork))

### 2. Make your data folder

Create an empty folder named **`HealthTracker`** in your home folder:

- **Mac:** `~/HealthTracker` (i.e. `/Users/<you>/HealthTracker`)
- **Windows:** `C:\Users\<you>\HealthTracker`

This is where all your data lives, as plain files.

### 3. Install the Filesystem extension and point it at that folder

1. In Claude/Cowork, open **Settings → Extensions** (a.k.a. Connectors).
2. Click **Browse extensions**, find the official **Filesystem** extension, and click **Install**.
3. When it asks which folder it may access, choose your **`HealthTracker`** folder.
4. Make sure it's **enabled for this chat** (the "+" → Connectors panel).
5. **Quit Claude desktop completely and reopen it** — the new extension only loads on restart.

After restart, type "list files in my HealthTracker folder" in Cowork. If Claude can do it,
you're set.

### 4. Allow network access to Oura's API

This step gets missed, and skipping it is the #1 reason the Oura sync fails silently.

1. Go to **Settings → Capabilities** (Team/Enterprise: your org's admin settings →
   **Capabilities**) → find the **Code execution** network/domain-allowlist control.
2. Either switch it to **"All domains"**, or add `api.ouraring.com` to **"Additional
   allowed domains"**.
3. **Start a brand-new Cowork chat after saving this** — it only applies to sessions
   created after the change.

Full detail and troubleshooting: `skills/health-log/references/oura-import.md`.

### 5. Install the plugin

In Cowork, type these two commands:

```
/plugin marketplace add pbrosen/brads-oura-health-tracker
/plugin install health-tracker-oura@brad-health-oura
```

### 6. Connect your folder

Point Cowork at your `~/HealthTracker` folder for the session, so chat-based logging can
read and write it too.

### 7. Open your dashboard

Say to Claude: **"Set up my Health Tracker dashboard."** Claude creates the dashboard
(a live page you can pin and reopen), wiring it to your installed Filesystem extension and
finding your data folder automatically.

### 8. Enter your numbers

The first time you open it, a short welcome wizard asks for your weight, height, age, and
typical activity level — this drives the resting-calorie math. You can change these anytime
in **⚙ Calorie-burn settings**.

### 9. Get your Oura token and create the daily import task

1. Go to https://cloud.ouraring.com/personal-access-tokens, log in, click **Create New
   Personal Access Token**, name it "Health Tracker", and copy the token (shown once).
2. Paste the task-creation prompt from `skills/health-log/references/oura-import.md` into
   Cowork chat, with your token filled in. **Name the task exactly `oura-daily-import`** —
   the dashboard's "↻ Import Oura" button depends on that exact name.
3. Approve the task. It also runs once immediately, so you should see today's Oura numbers
   right away.

That's it — you're running.

---

## Day-to-day use

**By chatting (works anywhere, even on mobile):**

- "I had two eggs and toast at 7:30 and a turkey sandwich for lunch."
- "Log 45 minutes of weight training, about 280 calories."
- "Log a bout of diarrhea this morning, Bristol 6, felt like it was the dairy."
- "What have I eaten so far today? How many calories left?"
- "What's my net calorie balance today?" *(includes today's Oura sync if it's run)*
- "Run my Oura import now" *(pulls a fresh number instead of waiting for tonight)*
- "What foods might be triggering my stomach?"

**In the dashboard:** add food (leave macros blank to let Claude estimate them), exercise,
and gut episodes; see net calories, macros, activity minutes, and a 7-day chart; click
**↻ Import Oura** to sync on demand; and click **🔎 Find food correlations** to scan which
foods precede your diarrhea/loose episodes. Logging by chat and in the dashboard write the
**same files**, so they stay in sync.

---

## Where your data lives

Inside your `HealthTracker` folder:

```
HealthTracker/
├── food-log/
│   ├── 2026-06-13.md        ← one file per day (food + manual exercise)
│   └── _library.md          ← your personal food database (auto-built)
├── symptom-log.md           ← rolling log of gut episodes
└── oura/
    └── 2026-06-13.json      ← one file per day, written by the nightly import task
```

Normal Markdown/JSON files — open, back up, or sync them however you like.

---

## Troubleshooting

- **"Couldn't find your Health Tracker folder."** Almost always means the Filesystem
  extension isn't installed/enabled, or it isn't pointed at your `HealthTracker` folder, or
  the folder doesn't exist. Re-check steps 2–3.
- **Dashboard shows but won't save.** The Filesystem extension didn't connect — confirm it's
  installed, enabled for this chat, and granted access to the folder. If saving still fails,
  the dashboard's connector name may differ on your machine; tell Claude "the dashboard can't
  save" and it can re-wire it.
- **No Oura data / the import task errors on api.ouraring.com.** See step 4 (network
  access) and `skills/health-log/references/oura-import.md` — this is almost always a missed
  domain-allowlist setting, or the task being created in a session that started before you
  saved that setting.
- **"↻ Import Oura" button errors immediately.** Your scheduled task probably isn't named
  exactly `oura-daily-import`. Rename or recreate it.

---

## Notes

- This is a personal tracker, **not medical advice**. The food↔symptom correlation is a
  rough statistical hint to discuss with a doctor or dietitian, not a diagnosis. "Activity
  minutes" from Oura is Oura's own metric, not a lab VO2max measurement.
- Everything stays on your computer except the direct Oura API calls the nightly task makes
  with your own token — the Filesystem extension reads/writes only the one `HealthTracker`
  folder you granted it.
- Questions or something looks off? Screenshot it and send it to Brad — he can adjust the
  parsing.
