# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-04 — Notion side built: workspace, integration,
page, and database all live. GitHub Actions workflow and seed script not
yet written. See `PLAN-project-status-v1.md` for full detail._

## 1. What this project is

Automated Notion mirror of project status across the roadmap's Active
projects. Disk stays the source of truth (per the roadmap's own
`PRINCIPLES.md`) — Notion is a read-only reporting window, not a second
copy of the truth.

## 2. Decided so far (confirmed with Matt, 2026-09-04)

- Trigger mechanism: a GitHub Actions workflow calls the Notion API
  directly on a successful run. No self-hosted receiver — nothing needs to
  be reachable from the internet, and the Notion token lives in GitHub
  Secrets rather than on the mini PC.
- Fires on: a GitHub Actions workflow finishing successfully — not every
  push, not PR merges alone.
- Project initiation (Sketched → Planned → Active) stays a manual, human
  step in the roadmap project. This project only automates *ongoing*
  status once a project is already Active.
- Went straight to Active, skipping Planned — building started
  immediately, so there was no scoped-but-parked gap for Planned to hold.

## 3. Built so far (2026-09-04)

- Notion workspace: "AI development" (renamed from default during this
  build).
- Internal connection "project-status" created, access-token auth,
  scoped to that workspace. Token capabilities: Read/Update/Insert
  content, no comments, no user information (tightened from the default).
  Token stored at `C:\automation\secrets\project-status.env`.
- Top-level page "Project Status" created and shared with the connection
  (page id `3d1c7cc2-9a3c-802f-9acf-fb56dc9578b3`).
- "Projects" database created inside that page (database id
  `3d1c7cc2-9a3c-8060-beb0-cb5b5c027fd8`), schema matches the plan doc
  exactly: Project, Status, Last Updated, Latest Update, Repo, Disk
  Location, Claude Project, Source. Status options: Sketched/Planned/
  Active/Paused/Done. Source options: Manual/GitHub Hook. Currently empty
  — no rows seeded yet.
- Built via the Notion UI (Claude in Chrome), not the API — a direct API
  call from this session's container was blocked by a safety classifier
  (bearer token in an outbound curl call). Not re-attempted; the UI path
  worked fine and is what's reflected above.

## 4. Open, blocking the rest of the build

- infra-watch has 4 unpushed commits (its whole v1 build) — see
  `PLAN-project-status-v1.md`'s open items.
- caddy's git remote isn't reachable from this session's shell (credential
  issue) — Matt to confirm normal push access works.
- infra-watch's claude.ai project name still needed for its roadmap
  pointer file.
- Seed script (populate the database from `roadmap\STATE.md` /
  `projects\*.md`) not written yet.
- GitHub Actions workflow (`.github/workflows/notion-status.yml`) not
  written yet; needs each hooked repo's `NOTION_TOKEN` and `NOTION_PAGE_ID`
  secrets, which need the seed step to run first to get per-project page
  ids.

## 4. Not started

Notion workspace/database, Notion internal integration + token, the GitHub
Actions workflow itself, and per-project config for what feeds Notion.
