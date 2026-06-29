# ansible-server-bootstrap

A single Ansible playbook that turns a fresh Debian/Ubuntu VPS into a production-ready server in one command.

## What it does

| Step | Details |
|------|---------|
| System hardening | `apt` upgrade, UFW firewall (allow 22/80/443, deny everything else), fail2ban, unattended-upgrades |
| nginx | Installs and enables nginx |
| Docker | Installs Docker Engine, CLI, Buildx, and the Compose plugin from the official Docker apt repo |
| GitHub Actions runner | Creates a dedicated `github` user, downloads the latest runner release, registers it against your repo, and installs it as a systemd service |

Tested on: **Ubuntu 22.04 / 24.04**, **Debian 12** — on Hetzner, DigitalOcean, and similar VPS providers.

---

## Prerequisites

- Ansible 2.12+ on your local machine (`pip install ansible` or `brew install ansible`)
- A fresh VPS reachable over SSH as `root`
- A GitHub PAT with runner registration rights:
  - **Classic PAT** → `repo` scope
  - **Fine-grained PAT** → Repository → Administration = Read & Write

---

## Quick start

```bash
ansible-playbook -i 'YOUR_SERVER_IP,' setup.yml -u root \
  -e github_repo=your-org/your-repo \
  -e runner_reg_token=AXXXXXXXXXXXXXXXXX
```

> The trailing comma after the IP is required — it tells Ansible to treat the value as an inline inventory.

---

## Variables

Edit the `vars` block at the top of `setup.yml`, or pass any variable on the command line with `-e key=value`.

| Variable | Default | Description |
|----------|---------|-------------|
| `github_repo` | `your-org/your-repo` | `owner/repo` the runner registers against |
| `runner_reg_token` | placeholder | Registration token from GitHub UI (expires in 1h) |
| `runner_name` | hostname | Name shown in the GitHub UI under **Settings → Runners** |
| `runner_labels` | `self-hosted,linux,vps` | Comma-separated labels for workflow targeting |
| `runner_user` | `github` | Dedicated OS user that owns the runner process |
| `runner_arch` | `x64` | Set to `arm64` for ARM-based VPS instances |

---

## Getting the registration token

Go to your repo → **Settings → Actions → Runners → New self-hosted runner**, then copy the token from the `--token` line in the Configure section. Tokens expire after **1 hour**.

## Keeping the token out of version control

Never commit a real token. Pass it at runtime instead:

```bash
ansible-playbook ... -e runner_reg_token=AXXXXXXXXXXX
```

---

## Re-running safely

All tasks are idempotent. Re-running the playbook against a server that was already bootstrapped will:
- Skip the runner registration step (detected via `~/.runner` sentinel file)
- No-op on packages and services that are already in the desired state

---

## License

MIT
