# Flow Kitsu — Deployment Changelog

Production deploy history for the Flow fork of Kitsu. Newest first.
Each entry records the deployed commit, the production backup taken at deploy
time, and the exact rollback command.

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
