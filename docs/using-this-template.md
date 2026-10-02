# Update procedure

Each service in this repo follows the same rhythm:

1. **Deploy** — copy (or move) the service's `.container`/`.env` into
   `~/.config/containers/systemd/`, then `systemctl --user daemon-reload`
   and `systemctl --user start <name>.service`.
2. **Declare backups** — add a block to `/services/backup/backup-services.yaml`
   for anything stateful, then re-run `homelab-backup install-timers`.
3. **Verify** — `systemctl --user start homelab-backup-<service>-<id>.service`,
   and check the dashboard for a green pill.

Notes:
- Pin images (digest or version tag) in production services; the example is loose on purpose.
- Quadlet `AutoUpdate=registry` + `podman-auto-update.timer` can be used for rolling updates.
- Secrets belong in per-service `.env` files, wired via `EnvironmentFile=` in the unit.
