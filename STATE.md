# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-04 — Notion side built and seeded: workspace,
integration, page, database, and all 3 rows (caddy, infra-watch,
project-status) live. Git push confirmed possible from this session's
device shell once credentials exist (network works; only auth was
missing — see section 4, in progress with Matt). GitHub Actions workflow
still not written. See `PLAN-project-status-v1.md` for full detail._

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

## 4. Git push — network works, only auth was missing

Corrected finding: the device shell *does* have outbound network access
(`curl https://github.com` and `https://api.github.com` both return 200;
`git ls-remote origin` succeeds against infra-watch). The earlier "no
network" read was wrong or stale. The actual and only blocker is
credentials: no `credential.helper` configured (local, global, or
system), no `gh` CLI, nothing on PATH to authenticate a push.

Fix in progress with Matt (2026-09-04, from the mini PC): he's creating a
GitHub fine-grained personal access token, scoped to just infra-watch,
pinball-caddy, and project-status, Contents: Read and write. It'll be
stored at `C:\automation\secrets\github.env` as `GITHUB_TOKEN=...`,
matching the existing per-project `.env` pattern in that folder. Once
present, each repo's local (not global) remote gets rewritten to
`https://x-access-token:<token>@github.com/...` so push works from this
shell without touching Matt's own terminal credentials.

Affects three repos:
- infra-watch: 4 local commits (its whole v1 build), unpushed.
- project-status: 3 local commits (today's build + doc work), unpushed.
- caddy: fetch also failed earlier under the same missing-auth condition;
  recheck once the token's in place.

## 5. Open, blocking the rest of the build

- Git push (section 4) — token creation in progress with Matt.
- GitHub Actions workflow (`.github/workflows/notion-status.yml`) not
  written yet; needs each hooked repo's `NOTION_TOKEN` and
  `NOTION_PAGE_ID` secrets. Page/database ids already exist (section 3)
  since seeding is done — no longer blocked on that.

## 6. Seeded rows (2026-09-04)

All via the Notion UI (browser automation), same reason as the schema
build — direct API blocked by a safety classifier in this session's cloud
container. Source set to Manual for all three (none are hooked yet):

| Project | Status | Last Updated | Repo |
|---|---|---|---|
| Pinball Caddy | Active | Sep 3, 2026 | pinball-caddy |
| Infra-Watch | Active | Sep 4, 2026 | infra-watch |
| Project Status | Active | Sep 4, 2026 | project-status |

Also fixed while seeding: the Status property was missing "Paused" (only
had Sketched/Planned/Active/Done, 4 of the intended 5 options). Added it
and reordered to Sketched/Planned/Active/Paused/Done.
