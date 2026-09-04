# PLAN-project-status-v1.md — what this project builds and why

## What this is

An automated Notion mirror of status for the roadmap's Active projects,
built on top of the roadmap project's own `STATE.md` / pointer-file system.
Disk stays the source of truth — the roadmap repo, and each project's own
`STATE.md` — Notion is a reporting window onto that, never a second copy
of it. Nothing written in Notion flows back to disk.

## Confirmed decisions (Matt, 2026-09-04)

- Trigger mechanism: a GitHub Actions workflow in each hooked project's own
  repo calls the Notion API directly. No self-hosted receiver — nothing
  needs to be reachable from the internet, and the Notion token lives in
  GitHub Secrets, not on the mini PC.
- What counts as an update, for infra-watch specifically: **development
  progress on the project itself** (new features — e.g. Discord
  notifications being added), not its weekly Task Scheduler run. The
  weekly risk-assessment run is infra-watch's own product; it isn't what
  this project reports on.
- Project initiation (Sketched → Planned → Active) stays a manual, human
  step in the roadmap project. This project only automates *ongoing*
  status once a project is already Active.
- Went straight to Active, skipping Planned — building started
  immediately, so there was no scoped-but-parked gap for Planned to hold.

## v1 scope

- A Notion database with one row per roadmap project, mirroring
  `roadmap\STATE.md`.
- Automated push-based updates wired for two repos to start:
  **infra-watch** and **caddy** (`pinball-caddy` on GitHub). Every other
  row is seeded once from `STATE.md` and stays manually refreshed until it
  gets its own hook.
- Other projects "come online" the same way, one at a time, as Matt
  decides to wire them.

## Notion database schema

| Property | Type | Notes |
|---|---|---|
| Project | Title | Matches the name in `roadmap\projects\*.md` |
| Status | Select (Sketched / Planned / Active / Paused / Done) | Same vocabulary as `STATE.md` |
| Last Updated | Date | Set by the hook on push, or by the seed script otherwise |
| Latest Update | Text | Commit subject line from the triggering push |
| Repo | URL | GitHub repo link, where one exists |
| Disk Location | Text | Local folder path, for projects without a GitHub repo yet |
| Claude Project | Text | Name of the linked claude.ai project |
| Source | Select (Manual / GitHub Hook) | Whether this row is kept fresh automatically or needs a manual nudge — visible at a glance |

## Trigger mechanism

A GitHub Actions workflow, `.github/workflows/notion-status.yml`, added to
each hooked repo. Fires on push to `main`. PATCHes one fixed Notion page ID
(stored as a repo secret, set once when that project's row is created)
with: Last Updated = now, Latest Update = the commit subject line, Source
= GitHub Hook.

Each hooked repo needs two GitHub secrets: `NOTION_TOKEN` (the integration
token — same one can be reused across repos, since it's scoped to one
database either way) and `NOTION_PAGE_ID` (that repo's own row).

## Setup steps

1. Matt creates a Notion integration at notion.so/my-integrations, scoped
   to his workspace — gets an internal integration token.
2. Matt creates the database by hand in Notion, using the schema above,
   and shares it with the integration (Notion requires this per-database;
   it doesn't happen automatically).
3. Once the database exists, Claude writes a one-time seed script that
   reads `roadmap\STATE.md` / `projects\*.md` and creates the initial rows
   — needs the database ID and token from Matt to run.
4. For each hooked repo (infra-watch, caddy): get that row's Notion page
   ID from the seed step, add it as `NOTION_PAGE_ID` in that repo's GitHub
   secrets, plus `NOTION_TOKEN`. Claude drops the workflow file in once
   secrets exist and confirms the first live push updates the row.

## Open items, as of 2026-09-04

- **infra-watch has 4 commits sitting unpushed to GitHub** — its entire v1
  build (`b336544` through `d82554e`). The hook can't see anything that
  hasn't been pushed; these need to go up before this is testable there.
- **caddy: git fetch/push failed from this session's shell** —
  `could not read Username for 'https://github.com'`. Likely a
  credential-cache difference in this execution context rather than a
  real problem, but worth Matt confirming he can push normally from his
  own terminal before this gets relied on.
- infra-watch's claude.ai project name — needed for its roadmap pointer
  file (`claude_project` field currently blank).
- Exact Notion API version header / field names get finalized once the
  database actually exists and Claude can see its real schema.
