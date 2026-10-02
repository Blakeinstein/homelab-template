# homelab-template

Starter layout for running self-hosted services as **Podman Quadlets**
(rootless systemd units) with a **declarative backup system**.

Use this template (`gh repo create --template` or "Use this template" on
GitHub) to bootstrap a homelab repo, then let
[**homelab-backup**](https://github.com/Blakeinstein/homelab-backup) back
everything up from a single YAML file.

## Layout

```
services/<service-name>/
├── <name>.container        # quadlet unit → ~/.config/containers/systemd/
├── <name>.env              # runtime config incl. credentials (gitignored!)
└── <name>.network          # optional per-service network
```

- Service data lives **outside the repo**, in a data root (e.g.
  `/srv/homelab-data/<service>/`), bind-mounted by the quadlet.
- Credentials live only in each service's `.env` — never committed.
  Put per-service paths in the agent `.env` instead.

## Backups (the part that makes this repo useful)

1. Deploy [homelab-backup](https://github.com/Blakeinstein/homelab-backup)
   on the host (binary + `homelab-backup.env`).
2. Declare every service + procedure in
   [`services/backup/backup-services.yaml`](services/backup/backup-services.yaml).
3. Generate the systemd user timers:

   ```sh
   homelab-backup install-timers
   ```

That is entire recurring-maintenance workflow. New service = new YAML block.

## Quick start

```sh
# 1. clone your copy of this template, add services under services/
git clone <your-fork-url> homelab && cd homelab

# 2. get the backup agent
gh repo clone Blakeinstein/homelab-backup && cd homelab-backup
cp .env.example homelab-backup.env    # edit paths to point at your homelab repo
go build -o bin/homelab-backup ./cmd/homelab-backup
./bin/homelab-backup init

# 3. declare services in ../homelab/services/backup/backup-services.yaml
#    (see examples in the README of homelab-backup)

# 4. schedule + dashboard
./bin/homelab-backup install-timers
cp systemd/homelab-backup.service ~/.config/systemd/user/
systemctl --user daemon-reload && systemctl --user enable --now homelab-backup
# dashboard: http://localhost:3095 (or your HOMELAB_BACKUP_PORT / _ADDR)
```

## Example service

See [`services/example/`](services/example/) for a minimal working unit to
copy from.

## License

MIT — see [LICENSE](LICENSE).
