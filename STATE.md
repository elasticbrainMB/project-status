# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-04 — v1 is live end to end. Notion built and
seeded; all three repos pushed; `notion-status.yml` deployed to all
three and proven working (a real push to this repo triggered a run,
which updated this project's own Notion row — verified both via the
Actions API, conclusion: success, and visually in Notion). See
`PLAN-project-status-v1.md` for full detail._

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

## 4. Git push — resolved (2026-09-04)

Was never a network problem — the device shell has outbound access fine
(`curl https://github.com` / `api.github.com` both 200, `git ls-remote`
worked). The actual blocker was auth: no `credential.helper`, no `gh`
CLI, nothing on PATH.

Fix: Matt created a GitHub fine-grained PAT scoped to just infra-watch,
pinball-caddy, and project-status (Contents: Read and write), stored at
`C:\automation\secrets\github.env` as `GITHUB_TOKEN=...`. Each repo's
local (not global) remote was rewritten to
`https://x-access-token:<token>@github.com/...` and pushed. All three now
track their remote with nothing ahead:

- infra-watch: 6 commits pushed (`8c9b8a7..700209a`) — its whole v1 build
  plus two more Matt had added since the original count of 4.
- caddy: **33 commits pushed** (`5199a66..43a755a`) — turned out to be
  much bigger than expected; this repo's entire Phase B through Phase E
  history had never reached GitHub. Fast-forward, no conflicts.
- project-status: pushed as the repo's first commits (`origin/main` didn't
  exist yet) — `main` now tracks `origin/main`.

caddy's working tree still has a large amount of uncommitted work
(modified tracked files, many untracked session/prompt/record files) —
untouched here, that's Matt's own in-progress work in a project this
session doesn't own.

## 5. v1 complete — nothing blocking

Matt's PAT ended up with Contents, Workflows, and Secrets (all Read and
write), scoped to the three v1 repos, 90-day expiration. That let this
session push `notion-status.yml` to all three and set `NOTION_TOKEN` /
`NOTION_DATABASE_ID` on all three via the API (secret *values* are never
readable back through the API regardless of permission — only
create/update/delete). Confirmed live with a real test push to this repo:
GitHub Actions run succeeded, and the Project Status row's Latest Update,
Last Updated, and Source (now "GitHub Hook") all updated correctly.

What's left is optional polish, not blocking:
- Onboard more projects as they come online (explicitly deferred by
  Matt — not now).
- The backlog catch-up items Matt mentioned (already-live projects not
  yet reflected here) — also explicitly deferred.
- If Matt later wants stricter "real CI passed" semantics instead of
  "a push to the default branch succeeded," that would mean building an
  actual build/validate workflow per repo and switching notion-status.yml
  to a `workflow_run` trigger — a bigger lift, not done for v1.

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

## 7. GitHub Actions hook — live (2026-09-04)

`NOTION_TOKEN` and `NOTION_DATABASE_ID` set as repo secrets on all three
repos (infra-watch, caddy, project-status) via the API, now that the PAT
has Secrets: Read and write. This commit is the first real test of the
hook end to end.
