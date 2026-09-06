# STATE.md — current state, fast orientation

**What this file is.** Current-facts-only summary — not a design document,
per the roadmap project's own convention. Read this first.

_Last updated: 2026-09-06 — v1 and Phase 2 (backlog sync) still live
end to end. Phase 3 added: a Description column, separate from Latest
Update. See `PLAN-project-status-v1.md` for the original build, section 8
for the backlog sync, section 9 for the Description column._

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


## 8. Backlog sync — Sketched and Planned rows (2026-09-05)

Second phase, requested by Matt after v1: mirror `BACKLOG.md` (Sketched
ideas) and `projects\*.md` pointer files with `status: planned` into the
same Notion database, so the dashboard shows the whole pipeline, not just
Active projects. Lives entirely in the roadmap repo, not here, since it
watches roadmap's own files rather than a single project's build:
`C:\automation\roadmap\.github\workflows\notion-backlog-sync.yml` +
`C:\automation\roadmap\scripts\notion_backlog_sync.py`.

**Decided with Matt before building (4 questions, all answered):**
- A graduating idea's pointer file must reuse its BACKLOG.md heading text
  exactly as its `name` field, so the sync updates the same Notion row
  instead of creating a second one on graduation. Documented in
  `roadmap\CLAUDE.md` and `roadmap\projects\_TEMPLATE.md`.
- Also watch `projects\*.md` files with `status: planned`, not just
  BACKLOG.md — so graduation shows up as a Status change on the same row.
- Added a "Dropped" option to Notion's Status property, for an idea
  abandoned outright (removed from BACKLOG.md, never graduates).
- Push-triggered, same shape as the three per-project hooks.

**Edge case found while designing, not covered by the four questions
above:** some ideas skip the pointer-file stage entirely — "Cycling-day
rating" and "vpin skill-building" both reached Active/Paused status with
no `projects\*.md` file, because the pointer-file system only kicks in
once there's a `disk_location` to point at (n8n-only or no-folder-yet
projects never get one). A naive "title missing from BACKLOG.md and
Planned files = Dropped" rule would have mislabeled those as abandoned
right at the moment they actually succeeded.

**Fix:** before marking anything Dropped, the script also checks whether
the title still appears anywhere in `STATE.md`'s own text. `STATE.md` is
supposed to mention every tracked project somewhere per the roadmap's own
sync discipline, so this is a cheap, effective safety net — confirmed
against Matt's own current STATE.md draft, which does mention
"Cycling-day rating" by that exact name. Cost of the safety net: a title
it protects also won't get its Notion row auto-corrected (e.g. Sketched
→ Active) — that stays a manual STATE.md-driven fix, same as before this
sync existed. Never touches a row whose Notion Status is already Active,
Paused, Done, or Dropped — those belong to the per-project hooks and to
Matt's own STATE.md edits.

**Reconciliation is a full state check on every run, not an incremental
diff of the commit** — simpler and more robust than trying to diff two
versions of BACKLOG.md, and consistent with "disk is the source of
truth": Notion for these rows is always a full re-derivation of whatever
BACKLOG.md and the Planned pointer files currently say.

**Built and pushed to the roadmap repo (commit `da42b79`):** the workflow,
the script, and the two doc updates only — Matt's own pending edits to
`BACKLOG.md`, `STATE.md`, and the `projects\*.md` files in that repo were
left exactly as he had them, uncommitted, per the roadmap's own practice
of leaving that project's changes for his review.

**First live run already happened, unintentionally** — the push above
touched `projects\_TEMPLATE.md`, which matched the workflow's own path
filter (`projects/**.md`) and fired it against the *old, still-committed*
BACKLOG.md and pointer files (Matt's newer versions are local-only until
he pushes). Result: six Sketched rows reflecting the stale backlog text,
plus one incorrect duplicate — a lowercase "infra-watch" Planned row,
because the old committed `projects\infra-watch.md` still says `status:
planned`. Deleted that duplicate by hand via Notion. The six stale
Sketched rows were left alone — they're harmless and self-correct the
next time Matt pushes his real BACKLOG.md/STATE.md/projects changes (the
Model-version-review and beehiiv rows will correctly flip to Dropped
since their old titles no longer appear anywhere current; Cycling-day
rating will correctly stay protected by the STATE.md check instead of
being dropped).

**Not yet done:** Matt hasn't pushed his pending roadmap changes, so the
sync hasn't run against real current content yet. That push is the next
real test.

## 9. Description column added (2026-09-06)

Matt asked for a dedicated Description column, separate from Latest
Update -- Latest Update had been doing double duty as both a static
"what is this project" summary and a dated "what just happened" status
line. Split the two:

- **Description** -- one or two plain-language sentences, seeded exactly
  once from disk (the pointer file's body text for Planned/Active/Paused/
  Done rows, the BACKLOG.md entry's text for Sketched rows) and then left
  alone by every script for good, so Matt can hand-polish wording in
  Notion without a later disk edit overwriting it.
- **Latest Update** -- unchanged for Active rows (still the commit
  subject line from each project's own `notion-status.yml` hook). For
  Sketched/Planned rows, `notion_backlog_sync.py` (in the roadmap repo)
  now only changes it on a real status move ("Moved to Planned on
  September 6, 2026") or first creation ("Added to roadmap on
  September 6, 2026"), instead of bumping to today on every push that
  touches any backlog file, even ones nothing happened to.

**Field ownership, made explicit:** `notion-status.yml` (this repo, and
caddy's, infra-watch's) owns Last Updated / Latest Update / Source for
Active rows and never touches Description. `notion_backlog_sync.py`
(roadmap repo) fully owns Sketched/Planned rows, and additionally seeds
Description-only (never Status/dates/Source) for Active/Paused/Done
pointer files it finds in `roadmap\projects\`, so an Active project's
Description can be set just by editing its own roadmap pointer file --
no changes needed in this repo or in caddy's/infra-watch's.

Schema and view updated directly via the Notion connector (no browser
automation needed this time -- the connector reaches the Notion API
directly from this session, unlike the earlier `curl`-with-bearer-token
path that a safety classifier blocked during the original v1 build).
All 12 existing rows were also backfilled with a Description directly
through the connector, so the dashboard reflects the split immediately
rather than waiting on the next push.

**Not done yet, by design:** `roadmap\projects\infra-watch.md` is still
committed with stale content (`status: planned`, and `name: "infra-watch"`
lowercase -- the same casing that caused a duplicate row once before, per
section 8's edge-case notes). Matt has a newer version of that file
sitting uncommitted locally, fixing the status but not the casing. The
roadmap-repo commit for this Description-column change was deliberately
built and committed *without* pushing, specifically to avoid triggering
`notion-backlog-sync.yml` against that stale committed file -- doing so
would recreate the exact duplicate-row bug fixed in section 8. Once Matt
commits his own infra-watch.md fix (ideally also correcting `name` to
"Infra-Watch" to match Notion's row exactly), both commits can be pushed
together safely.
