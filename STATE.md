# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-04 — Notion side built: workspace, integration,
page, and database all live. Git push from this session's device shell
confirmed not viable (see section 4). GitHub Actions workflow and seed
script not yet written. See `PLAN-project-status-v1.md` for full detail._

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
  call from this session's cloud container was blocked by a safety
  classifier (bearer token in an outbound curl call). Not re-attempted;
  the UI path worked fine and is what's reflected above.

## 4. Git push — confirmed not possible from this session's device shell

Checked directly: no `credential.helper` configured (local, global, or
system), no `gh` CLI, no credential-manager binary on PATH in this
session's device shell. That shell is an isolated Linux VM this session
uses to reach the mini PC's mounted folders — it's separate from Matt's
normal Windows terminal, which is presumably where his existing commits on
these repos came from and where his real git credentials live. This isn't
a bug to fix in this session; pushing has to happen from Matt's own
terminal, or he tells me a different way to authenticate.

Affects three repos:
- infra-watch: 4 local commits (its whole v1 build), unpushed.
- project-status: 2 local commits (today's STATE.md/plan work), unpushed.
- caddy: can't even fetch from this shell, so push state is unknown.

This blocks the GitHub Actions hook entirely — a workflow can't finish
successfully on a repo GitHub doesn't have current code for.

## 5. Open, blocking the rest of the build

- Git push (section 4) — needs Matt.
- caddy's current push/fetch status — needs Matt to confirm.
- Seed script (populate the database from `roadmap\STATE.md` /
  `projects\*.md`) not written yet.
- GitHub Actions workflow (`.github/workflows/notion-status.yml`) not
  written yet; needs each hooked repo's `NOTION_TOKEN` and
  `NOTION_PAGE_ID` secrets, which need the seed step to run first to get
  per-project page ids.
