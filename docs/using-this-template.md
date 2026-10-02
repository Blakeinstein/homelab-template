# Update procedure

Each service in this repo follows the same rhythm:

1. **Deploy** — copy (or move) the service's `.container`/`.env` into
   `~/.config/containers/systemd/`, then `systemctl --user daemon-reload`
   and `systemctl --user start <name>.service`.
2. **Declare backups** — add a block to `/services/homelab-backup/backup-services.yaml`
   for anything stateful, then re-run `homelab-backup install-timers`.
3. **Verify** — `systemctl --user start homelab-backup-<service>-<id>.service`,
   and check the dashboard for a green pill.
4. **(Optional) Label it for Homeio** — drop an `app.yaml` next to the unit
   (see [`services/example/app.yaml`](../services/example/app.yaml)) so it
   shows up named and iconed instead of as a generic container.

Notes:
- Pin images (digest or version tag) in production services; the example is loose on purpose.
- Quadlet `AutoUpdate=registry` + `podman-auto-update.timer` can be used for rolling updates.
- Secrets belong in per-service `.env` files, wired via `EnvironmentFile=` in the unit.
- `app.yaml` is read by Homeio, not by homelab-backup or systemd — it's purely dashboard metadata and safe to omit.
