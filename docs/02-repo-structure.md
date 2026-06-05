# 02 — Repo Structure

How to organize an infra config repo so Claude can understand it, and so you can deploy from it safely.

The reference implementation is [a777ance/localDNS](https://github.com/a777ance/localDNS).

---

Proceed to [03 — GitHub setup](03-github-setup.md).

---

## SETUP.md

This is the "new machine" guide. If your server dies today, SETUP.md is what rebuilds it.
Write it as you set things up — don't reconstruct it from memory later.

Every commit to `main` should leave SETUP.md able to reproduce a working system from scratch.

---

## What NOT to put in the repo

- Private keys (WireGuard, SSH) — use placeholders or secrets management
- Passwords or tokens — use environment variables or `.env` files that are gitignored
- System state files (`/etc/hosts`, `/etc/resolv.conf`) — these change constantly

Add a `.gitignore`:
```
*.key
*.pem
.env
secrets/
```

---

## Writing CLAUDE.md

CLAUDE.md is a briefing document, not a README. It should tell Claude:

1. **What this repo is** — one paragraph, be specific
2. **Hardware** — what machine, what OS, what NIC
3. **Network topology** — ASCII diagram if helpful
4. **Services table** — service, runtime, ports, who can reach it
5. **Deploy paths table** — repo path → system path → reload command
6. **Known issues** — things that are broken or need attention
7. **Verification commands** — how to confirm the system is healthy

Copy the template: [templates/CLAUDE.md.template](../templates/CLAUDE.md.template)

A real example: [a777ance/localDNS CLAUDE.md](https://github.com/a777ance/localDNS/blob/main/CLAUDE.md)

---

## The deploy paths table

In your `CLAUDE.md`, always include a deploy paths table. This is the most important thing Claude reads:

```markdown
| Repo path | System path | Reload command |
| --------- | ----------- | -------------- |
| `unbound/server.conf` | `/etc/unbound/unbound.conf.d/server.conf` | `sudo systemctl restart unbound` |
| `pihole/docker-compose.yml` | `~/pihole/docker-compose.yml` | `cd ~/pihole && docker compose up -d` |
```

When Claude sees this table, it knows exactly where each file lives and what to run after changing it. Without it, Claude has to guess.

---

## Folder layout

Mirror your system's directory hierarchy. Group by service, not by file type.

```
my-homelab/
├── CLAUDE.md                   ← required; tells Claude what this repo is
├── README.md                   ← human summary
├── SETUP.md                    ← full reproduction guide (new machine → working system)
├── unbound/
│   ├── server.conf             → /etc/unbound/unbound.conf.d/server.conf
│   └── tuning.conf             → /etc/unbound/unbound.conf.d/tuning.conf
├── pihole/
│   └── docker-compose.yml      → ~/pihole/docker-compose.yml
├── wireguard/
│   └── wg0.conf                → /etc/wireguard/wg0.conf
├── systemd/
│   ├── my-service.service      → /etc/systemd/system/my-service.service
│   └── my-service.d/
│       └── override.conf       → /etc/systemd/system/my-service.d/override.conf
├── scripts/
│   └── my-script               → /usr/local/bin/my-script
└── ufw/
    └── setup.sh                ← run directly to configure firewall
```

See the actual layout at: [github.com/a777ance/localDNS](https://github.com/a777ance/localDNS)

---

## The core idea

Every file in your repo maps 1:1 to a location on your live system. The repo is a snapshot; the server is the source of truth. Edits here don't take effect until you manually deploy (or a GitHub Action does it for you).
