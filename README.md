# homelab-template

Starter layout for running self-hosted services as **Podman Quadlets**
(rootless systemd units), with a **declarative backup system** and an
optional **dashboard**.

Use this template (`gh repo create --template` or "Use this template" on
GitHub) to bootstrap a homelab repo, then let
[**homelab-backup**](https://github.com/Blakeinstein/homelab-backup) back
everything up from a single YAML file and
[**Homeio**](https://github.com/doctor-io/homeio) show it all on a desktop.

## Layout

```
services/<service-name>/
├── <name>.container        # quadlet unit → ~/.config/containers/systemd/
├── <name>.env              # runtime config incl. credentials (gitignored!)
├── <name>.network          # optional per-service network
└── app.yaml                # optional: how Homeio labels this service
```

- Service data lives **outside the repo**, in a data root (e.g.
  `/srv/homelab-data/<service>/`), bind-mounted by the quadlet.
- Credentials live only in each service's `.env` — never committed.
  Put per-service paths in the agent `.env` instead.
- `services/homelab-backup/` is reserved: it holds `backup-services.yaml`,
  not a quadlet unit (homelab-backup runs as its own systemd service, not a
  container). Keep that exact folder name — both homelab-backup's own README
  and Homeio's auto-detection expect it.

## Backups (the part that makes this repo useful)

1. Deploy [homelab-backup](https://github.com/Blakeinstein/homelab-backup)
   on the host (binary + `homelab-backup.env`).
2. Declare every service + procedure in
   [`services/homelab-backup/backup-services.yaml`](services/homelab-backup/backup-services.yaml).
3. Generate the systemd user timers:

   ```sh
   homelab-backup install-timers
   ```

That is entire recurring-maintenance workflow. New service = new YAML block.

## Homeio dashboard (optional)

[Homeio](https://github.com/doctor-io/homeio) can discover every service in
this repo and show it on its desktop:

1. Point Homeio's `QUADLET_SERVICES_ROOT` environment variable at this repo's
   `services/` folder (wherever it's checked out on the host Homeio runs on).
2. Each subfolder with a `*.container` unit becomes an app card, resolved
   against the container actually running. Add an `app.yaml` next to the
   unit(s) — see [`services/example/app.yaml`](services/example/app.yaml) —
   to give it a name, icon, description, category, and web UI port instead of
   a generic container tile.
3. If you also run homelab-backup, turn it on in Homeio's
   **Settings → Integrations → homelab-backup** to get a Backups widget
   linking to its dashboard. Homeio finds its config automatically as long as
   it's at `services/homelab-backup/backup-services.yaml`, per the layout
   above.

Homeio never drives any of this (no start/stop/update) — it's read-only
labeling and status on top of what `homelab-backup` and your quadlets already
do.

## Quick start

```sh
# 1. clone your copy of this template, add services under services/
git clone <your-fork-url> homelab && cd homelab

# 2. get the backup agent
gh repo clone Blakeinstein/homelab-backup && cd homelab-backup
cp .env.example homelab-backup.env    # edit paths to point at your homelab repo
go build -o bin/homelab-backup ./cmd/homelab-backup
./bin/homelab-backup init

# 3. declare services in ../homelab/services/homelab-backup/backup-services.yaml
#    (see examples in the README of homelab-backup)

# 4. schedule + dashboard
./bin/homelab-backup install-timers
cp systemd/homelab-backup.service ~/.config/systemd/user/
systemctl --user daemon-reload && systemctl --user enable --now homelab-backup
# dashboard: http://localhost:3095 (or your HOMELAB_BACKUP_PORT / _ADDR)

# 5. (optional) point Homeio at ../homelab/services via QUADLET_SERVICES_ROOT
#    to see everything above on its dashboard too
```

## Example service

See [`services/example/`](services/example/) for a minimal working unit to
copy from, including an `app.yaml` showing the Homeio dashboard fields.

## License

MIT — see [LICENSE](LICENSE).
