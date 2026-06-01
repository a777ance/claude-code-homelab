# Claude Code SuperUser Setup (homelab)

A complete, opinionated guide to setting up Claude Code in VS Code for homelab and infrastructure config repos.

Built from real experience running [localDNS](https://github.com/a777ance/localDNS) — a self-hosted DNS + VPN + monitoring stack on an HP t630 thin client.

---

## What you will have at the end

- Claude Code running inside VS Code, connected to GitHub
- A config/infra repo with a proper folder structure Claude understands
- GitHub Actions deploy pipeline (push → auto-deploy to your server)
- A `CLAUDE.md` that makes Claude genuinely useful for your specific stack
- Best practices: branch strategy, secrets handling, SSH keys, syncing

---

## Steps

### 1 — Prerequisites

- VS Code with Claude Code extension installed
- Git configured: `git config --global user.name` / `user.email`
- GitHub account with SSH key added
- Claude Code authenticated: `claude login`

### 2 — Repo structure

Use the MANIFEST + `memory/` layout (see `memory/manifest-convention.md`).

Minimum viable structure for an infra repo:

```
your-repo/
├── CLAUDE.md           ← brief Claude, mention memory system
├── MANIFEST.md         ← paged memory index
├── README.md           ← human intro
├── memory/             ← all docs/context
└── configs/            ← actual config files (dnsmasq, wg, systemd, etc.)
```

### 3 — GitHub setup

- Generate SSH key: `ssh-keygen -t ed25519 -C "your@email.com"`
- Add to GitHub: Settings → SSH and GPG keys
- Set remote: `git remote set-url origin git@github.com:user/repo.git`
- Branch strategy: `main` = stable, feature branches for changes, Claude works on `claude/*` branches

### 4 — Deploy pipeline (GitHub Actions → SSH → reload)

See `memory/templates.md` for the full `deploy.yml`.

How it works:
1. Push to `main`
2. GitHub Actions SSHes to your server
3. Runs `git pull` + `systemctl reload` (or `docker compose up -d`)

Secrets needed in GitHub repo settings:
- `SSH_PRIVATE_KEY` — deploy key (read-only recommended)
- `SSH_HOST` — your server IP or hostname
- `SSH_USER` — deploy user

### 5 — Best practices

- Never commit secrets — use `.gitignore` (see templates)
- Keep `CLAUDE.md` updated as your stack changes
- One service per branch when making changes
- Tag stable releases: `git tag v1.0 && git push --tags`
- MANIFEST summaries are documentation — write them for future-you

---

## Case study

Every step above reflects how [a777ance/localDNS](https://github.com/a777ance/localDNS) is set up.
The `CLAUDE.md` in that repo is what a mature, Claude-aware infra repo looks like after real use.
