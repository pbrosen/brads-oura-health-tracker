# Connectors

This plugin uses two things: a filesystem connector (for your data) and direct calls to
Oura's API (for your daily activity — there's no Oura entry in Cowork's connector directory).

## 1. Local files — the Filesystem extension (you install it once)

The dashboard saves to your computer through the official **Filesystem desktop extension**.
It's a one-click install from Claude's **Settings → Extensions** directory and is fully
self-contained — it bundles its own runtime, so there's **no Node, no terminal, and no
config files** to deal with.

During install, point it at **one folder** — your `~/HealthTracker` folder — and that's
where all your data lives, as plain Markdown (and one JSON file per day) files. Nothing
leaves your computer except the direct Oura API calls described below. See `README.md`
(steps 1–2) for the click-through.

> **Tool-name note (for Claude at setup):** the dashboard talks to the Filesystem extension
> via `window.cowork.callMcpTool(...)`. The exact tool prefix the extension registers under
> can vary by machine/version. When the setup skill creates the dashboard, it sets the
> dashboard's `FS_PREFIX` (and the artifact's `mcp_tools`) to match the actual connected
> filesystem tools (e.g. `read_text_file` / `write_file` / `list_directory` /
> `read_multiple_files` / `create_directory` / `list_allowed_directories`). The default
> assumes the connector is named `filesystem`.

## 2. Oura (direct API calls, via a scheduled task — not a Cowork connector)

| Category        | Placeholder   | What it actually is                       |
| ---------------- | ------------- | ------------------------------------------ |
| Fitness / activity | `~~fitness` | **Oura** — direct REST API, no Cowork connector exists for it |

There's no off-the-shelf Oura connector, so a scheduled task calls
`https://api.ouraring.com/v2/usercollection/` directly with your personal access token and
writes one JSON file per day into `HealthTracker/oura/YYYY-MM-DD.json`. The dashboard (and
chat) read that file — Oura is **not** read live the way Strava is in Brad's own build.

**Before setting the task up, you must allow outbound network access to `api.ouraring.com`**
in **Settings → Capabilities → Code execution** (domain allowlist), in a session started
*after* you save that setting. Skipping this is the most common setup failure — see
`skills/health-log/references/oura-import.md` for the exact steps, the task prompt to paste,
and troubleshooting. The dashboard works fine before this is set up; it just won't have an
Oura number until the first successful sync.
