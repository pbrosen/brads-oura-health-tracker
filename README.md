# Brad's Health Tracker — Oura build

A Cowork **plugin marketplace** + source repo for a Health Tracker system fed by an
**Oura ring** instead of Strava: log food, exercise, and digestion (gut symptoms) to plain
local Markdown files, with a visual dashboard and a nightly Oura sync.

This is a sibling of [brads-strava-health-tracker](https://github.com/pbrosen/brads-strava-health-tracker)
(Brad's Strava-based friend build) — same dashboard and logging skill, forked to read Oura's
API instead of a live Strava connector.

## Install (for friends)

In Cowork / Claude Code:

```
/plugin marketplace add pbrosen/brads-oura-health-tracker
/plugin install health-tracker-oura@brad-health-oura
```

Then follow `health-tracker-oura/README.md` for the full setup — in particular, don't skip
the **network access** step (Settings → Capabilities → Code execution) before creating the
Oura import task, or the sync will fail silently.

## What's in here

```
.
├── .claude-plugin/
│   └── marketplace.json        # the marketplace manifest (lists the plugin below)
└── health-tracker-oura/        # the installable plugin (friend-facing build)
    ├── .claude-plugin/plugin.json
    ├── README.md                # full setup instructions
    ├── CONNECTORS.md            # Filesystem extension + Oura API notes
    └── skills/health-log/
        ├── SKILL.md             # logging + Oura-reporting skill
        └── references/
            ├── dashboard.html       # the dashboard artifact source
            └── oura-import.md       # Oura token + scheduled-task setup (with the network-access fix)
```

## Versioning

- Bump `version` in `health-tracker-oura/.claude-plugin/plugin.json` (and the matching entry
  in `marketplace.json`) each time you ship a change. Use semver: `0.1.0` → `0.1.1` for fixes,
  `0.2.0` for features.
- Tag releases in git: `git tag v0.1.0 && git push --tags`.
