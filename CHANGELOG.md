# Flow Kitsu — Deployment Changelog

Production deploy history for the Flow fork of Kitsu. Newest first.
Each entry records the deployed commit, the production backup taken at deploy
time, and the exact rollback command.

## 2026-07-30 — Upstream sync to CGWire 1.0.55

- **Deployed commit:** `b72a7365218aeabcbbbd82194d52250c785fbb1b` (`flow/main`)
- **Tag:** `kitsu-upgrade-20260730-133539`
- **Version:** `1.0.48` → **`1.0.55`** (upstream `cgwire/kitsu` latest)
- **Production backup:** `/opt/kitsu/releases/upgrade-20260730-133539-PREVIOUS`
- **Summary:** Merged upstream CGWire Kitsu into `flow/main`. Upstream touched all
  three files carrying the Flow `timesheet-orphan-hours` change
  (`Todos.vue`, `TimesheetList.vue`, `en.js`); all auto-merged and each Flow hunk
  was verified present afterwards (`otherTasks` prop + `other-tasks` binding,
  the `unassigned-row` tbody, and the `timesheets.unassigned_tasks` /
  `unassigned_hint` locale keys). Upstream added its own `task.unassigned_tasks`
  key — different namespace from Flow's `timesheets.unassigned_tasks`, so no
  collision. Only merge conflict was `CLAUDE.md`, resolved by keeping the Flow
  fork-workflow doc (same resolution as the 1.0.48 sync).
- **Build:** Node v22.23.1 (repo now requires **≥22.22.2**; system Node is v20, so
  a throwaway Node 22 was used for the build only). `npm ci && npm run build` clean;
  eslint clean via lint-staged.
- **Tests:** 1232/1233 unit tests pass. The single failure
  (`tests/unit/lib/time.spec.js`) is a pre-existing upstream test that hardcodes a
  Europe/Paris assumption — it passes under `TZ=Europe/Paris`, and both the test and
  `src/lib/time.js` are byte-identical to upstream v1.0.55. Not Flow-caused, not a
  product bug.
- **Scope:** Frontend only. Deployed alongside the Zou 1.0.52 → 1.0.62 bump the same
  day (kept as a matched pair).
- **Rollback:**
  ```bash
  sudo bash -c 'rsync -a --delete /opt/kitsu/releases/upgrade-20260730-133539-PREVIOUS/ /opt/kitsu/dist/ && chown -R zou:zou /opt/kitsu/dist'
  ```

## 2026-07-01 — Upstream sync to CGWire 1.0.48

- **Deployed commit:** `e6be73d1036297cf57340c2b79ec598a530dbeec` (`flow/main`)
- **Tag:** `kitsu-20260701-174502`
- **Version:** `1.0.21` → **`1.0.48`** (upstream `cgwire/kitsu` latest)
- **Production backup:** `/opt/kitsu/releases/kitsu-20260701-174502-PREVIOUS`
- **Summary:** Merged upstream CGWire Kitsu into `flow/main` to bring the fork to
  the latest published release (1.0.48). The Flow `timesheet-orphan-hours`
  customization was preserved through the merge (auto-merged cleanly; one prettier
  formatting fix). Only merge conflict was `CLAUDE.md` (upstream now ships its own
  dev guide) — resolved by keeping the Flow fork-workflow doc. Built with Node
  v22.23.1 (repo requires ≥22.22.1; system Node is v20, so a local Node 22 was
  used for the build only).
- **Scope:** Frontend only. Deployed alongside the Zou 1.0.28 → 1.0.52 bump the
  same day (kept as a matched pair).
- **Rollback:**
  ```bash
  sudo bash -c 'rsync -a --delete /opt/kitsu/releases/kitsu-20260701-174502-PREVIOUS/ /opt/kitsu/dist/ && chown -R zou:zou /opt/kitsu/dist'
  ```

## 2026-05-27 — Timesheet orphan hours

- **Deployed commit:** `60928abad52a48a670eb074ca5bd233640fff4a9`
- **Tag:** `kitsu-deploy-20260527-timesheet-orphan-hours`
- **Production backup:** `/opt/kitsu/releases/dist-backup-20260527-124135`
- **Summary:** Artist timesheets now show historical hours for tasks the artist
  is no longer assigned to, under a read-only **"Other logged tasks (view only)"**
  section. Previously those saved hours were hidden from the artist's own view
  even though the data was intact and still visible in the manager's production
  timesheet.
- **Scope:** Frontend only. No Zou / backend / plugin changes.
- **Rollback:**
  ```bash
  sudo bash -c 'rsync -a --delete /opt/kitsu/releases/dist-backup-20260527-124135/ /opt/kitsu/dist/ && chown -R zou:zou /opt/kitsu/dist'
  ```
