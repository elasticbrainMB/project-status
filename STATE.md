# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-04 — scope and architecture confirmed with Matt;
`PLAN-project-status-v1.md` is now authoritative on how and why. Nothing
built yet beyond this scaffold._

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

## 3. Open, blocking the build

- infra-watch has 4 unpushed commits (its whole v1 build) — see
  `PLAN-project-status-v1.md`'s open items.
- caddy's git remote isn't reachable from this session's shell (credential
  issue) — Matt to confirm normal push access works.
- Notion database doesn't exist yet — Matt needs to create the
  integration and the database (steps in the plan doc).
- infra-watch's claude.ai project name still needed for its roadmap
  pointer file.

## 4. Not started

Notion workspace/database, Notion internal integration + token, the GitHub
Actions workflow itself, and per-project config for what feeds Notion.
