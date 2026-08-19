# CLAUDE.md — Flow fork of **Kitsu** (frontend)

> Persistent workflow + deployment rules for this repo. Read this before editing,
> building, or deploying. Sister doc: the Flow fork of **Zou** (`flow-dev/zou/CLAUDE.md`).

## 1. Repo purpose
Flow Animation's customized fork of CGWire **Kitsu** (Vue 3 SPA, the web UI for the
Zou API). We layer Flow-specific UI changes on top of upstream Kitsu while staying
able to pull future CGWire releases.

## 2. Remotes (already configured — verify, don't recreate)
```
origin    https://github.com/flow-felix/kitsu.git     # Flow fork (our changes)
upstream  https://github.com/cgwire/kitsu.git         # CGWire (read-only source of truth)
```
- Fork parent on GitHub = `cgwire/kitsu`. Push only to `origin`. **Never push to `upstream`.**
- If org migration happens, `origin` becomes `github.com/FlowAnimationStudios/kitsu` (see migration plan). Keep `upstream` = cgwire.

## 3. Branch strategy
- `main` — tracks **upstream** Kitsu. Keep it a clean mirror of CGWire so syncing is conflict-free. Do **not** commit Flow changes here.
- `flow/main` — **integration branch**: upstream `main` + all Flow customizations. This is what production builds from. (To be created — see migration plan; today Flow edits live only in the working tree.)
- `flow/<feature>` — one branch per Flow change (e.g. `flow/timesheet-orphan-hours`), branched off `flow/main`, merged back via PR.
- `release/<tag>` — optional, cut from `flow/main` for a deploy.

## 4. Flow customizations — isolation rules
**Goal: touch as few upstream files as possible so upstream merges stay clean.**
Order of preference:
1. **Plugin** (`/flow-srv/zou/plugins/<name>/frontend`) — preferred for self-contained features (tickets, whiteboard, flowly, client-portal already do this).
2. **New component/file** imported at a single upstream injection point.
3. **Additive locale keys** (`src/locales/en.js`) — append, don't reorder.
4. **Minimal edit to an existing upstream file** — only when 1–3 can't work. Wrap with a comment marker:
   ```js
   // FLOW: <reason> — keep diff minimal for upstream merges
   ```
Each Flow change must be a discrete `flow/<feature>` branch so it can be reverted or rebased independently.

### Current Flow changes in this repo
- **`timesheet-orphan-hours`** — read-only "Other logged tasks" section so artists
  see hours logged on tasks they're no longer assigned to (`Todos.vue`,
  `TimesheetList.vue`, `en.js`). **Committed** on `flow/main` (branch
  `flow/timesheet-orphan-hours`, deployed 2026-05-27); survived the 1.0.48 merge
  on 2026-07-01, the 1.0.55 merge on 2026-07-30 and the 1.0.56 merge on 2026-08-19.
  Risk: medium (edits upstream files — upstream has touched all three in each of the
  last three syncs, so re-verify the hunks after every merge). Quick check: the
  `orphanIds` block in `Todos.vue` (3 references) and the
  `timesheets.unassigned_tasks` subtitle in `TimesheetList.vue`.
  - Note: upstream added its own `task.unassigned_tasks` locale key. Flow's key is
    `timesheets.unassigned_tasks` — different namespace, no collision. Don't
    "deduplicate" them.

## 5. Production paths (exact)
| Thing | Path |
|---|---|
| Source repo | `/home/felix-eyal/flow-dev/Kitsu-Mods/kitsu` |
| Deployed build (served) | `/opt/kitsu/dist` (dist-only, no git, owned `zou:zou`) |
| nginx site | `/etc/nginx/sites-available/zou` → enabled at `/etc/nginx/sites-enabled/zou` (`root /opt/kitsu/dist`; proxies `/api` → `127.0.0.1:5000`) |
| Domain | `https://kitsu.flowanimation.com` |
| Backups (proposed) | `/opt/kitsu/releases/<tag-or-timestamp>/` |

There is **no systemd unit** for the frontend — it is static files served by nginx. Restarting Zou does not affect it.

## 6. Build steps
> **Node version:** Kitsu 1.0.55 requires **Node ≥ 22.22.2** (`package.json`
> `engines`; Vite 8 toolchain). The prod box's system `node` is **v20** — building
> with it fails/produces a bad bundle. Use a Node 22 without touching the system:
> download `node-v22.x-linux-x64` to a temp dir and prefix `PATH` for the build
> only (`export PATH=/path/to/node-v22.../bin:$PATH`), or use nvm/fnm if installed.
> The 2026-07-01 (1.0.48) and 2026-07-30 (1.0.55) builds used a throwaway Node
> v22.23.1 this way; the 2026-08-19 (1.0.56) build used v22.23.2. `package.json`
> also requires **npm ≥ 10** — the bundled npm 10.9.8 satisfies it.
>
> **Unit tests:** `tests/unit/lib/time.spec.js` has an upstream test that hardcodes
> a Europe/Paris assumption and fails on this box (TZ `America/New_York`). Run
> `TZ=Europe/Paris npm run test:unit` to get a clean pass. Not a Flow regression.
> On 1.0.56 that yields a full **1334/1334 across 123 files**.

```bash
cd /home/felix-eyal/flow-dev/Kitsu-Mods/kitsu
git checkout flow/main          # always build from the Flow integration branch
npm ci                          # clean install from package-lock
npm run build                   # outputs to ./dist
# (optional) npm run lint && npm run test:unit
```

## 7. Deployment workflow (no direct editing of /opt/kitsu/dist)
```bash
# 1. Build (section 6) on a clean, committed tree
git status --porcelain          # MUST be empty
TAG="kitsu-$(date +%Y%m%d-%H%M%S)"
git tag "$TAG" && git push origin "$TAG"

# 2. Back up the currently-deployed build
sudo rsync -a --delete /opt/kitsu/dist/ "/opt/kitsu/releases/$TAG-PREVIOUS/"

# 3. Publish new build
sudo rsync -a --delete dist/ /opt/kitsu/dist/
sudo chown -R zou:zou /opt/kitsu/dist

# 4. Verify
curl -sI https://kitsu.flowanimation.com | head -1
```
nginx serves static files; no reload needed unless the nginx config itself changed (`sudo nginx -t && sudo systemctl reload nginx`).

## 8. Rollback workflow
```bash
# Fast rollback to the build saved at deploy time:
sudo rsync -a --delete /opt/kitsu/releases/<TAG>-PREVIOUS/ /opt/kitsu/dist/
sudo chown -R zou:zou /opt/kitsu/dist
# Or rebuild a known-good tag:
git checkout <good-tag> && npm ci && npm run build && (deploy as in §7)
```

## 9. Sync upstream safely
> **Fast-forward `main` to a release tag, not `upstream/main`.** Upstream `main`
> carries unreleased work — at the 2026-08-19 sync it was 85 commits past `v1.0.56`
> with an untagged per-project-roles feature that pairs with a matching Zou change.
> Taking `upstream/main` would half-deploy that across the two forks.

```bash
git fetch upstream --tags
git checkout main && git merge --ff-only v<x.y.z> && git push origin main
git checkout flow/main && git merge main         # resolve conflicts here, never on main
# build + full test checklist before deploying
```
Prefer `merge` (not rebase) on `flow/main` once it is shared, to keep history stable. Rebase only private/unpushed `flow/<feature>` branches.

## 10. Testing checklist (before any deploy)
- [ ] `git status` clean; built from a tagged commit on `flow/main`
- [ ] `npm run build` succeeds with no new warnings
- [ ] `npm run lint` clean on changed files
- [ ] Manual smoke: login, project list, **timesheets (assigned + orphaned-task read-only rows)**, task detail, playlist
- [ ] No console errors in browser
- [ ] Backup of current `/opt/kitsu/dist` taken (§7 step 2)

## 11. Commit standards
- Match upstream Kitsu style: `[area] short imperative summary` (e.g. `[timesheets] show read-only hours for unassigned tasks`).
- One logical change per commit; Flow-only commits live on `flow/*` branches.
- Reference the Flow issue/context in the body.

## 12. Safe-editing rules (hard rules)
1. **Never edit `/opt/kitsu/dist` directly** except a declared emergency hotfix — and even then, immediately back-port the change to the repo and redeploy from git.
2. Never commit Flow changes to `main` (the upstream mirror).
3. Keep working tree committed; production must be reproducible from a git tag.
4. Append-only for `locales/*` and shared config.
