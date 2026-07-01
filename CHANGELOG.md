# Flow Kitsu — Deployment Changelog

Production deploy history for the Flow fork of Kitsu. Newest first.
Each entry records the deployed commit, the production backup taken at deploy
time, and the exact rollback command.

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
