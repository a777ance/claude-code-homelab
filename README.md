# Claude Code SuperUser Setup (homelab)

A complete, opinionated guide to setting up Claude Code in VS Code for homelab and infrastructure config repos — the way a power user would, not the way the docs assume you will. Assumes you're comfortable in a terminal but new to Claude Code.

Built from real experience running [localDNS](https://github.com/a777ance/localDNS) — a self-hosted DNS + VPN + monitoring stack on an HP t630 thin client. That repo is the living case study throughout this guide.

---

## What you will have at the end

- Claude Code running inside VS Code, connected to GitHub
- A config/infra repo with a proper folder structure Claude understands
- GitHub Actions deploy pipeline (push → auto-deploy to your server)
- A `CLAUDE.md` that makes Claude genuinely useful for your specific stack
- Best practices: branch strategy, secrets handling, SSH keys, syncing

---

## Steps

| # | Guide | What it covers |
|---|-------|----------------|
| 1 | [Prerequisites](docs/01-prerequisites.md) | VS Code, Git, GitHub, Claude Code extension |
| 2 | [Repo structure](docs/02-repo-structure.md) | Folder layout, deploy paths, CLAUDE.md |
| 3 | [GitHub setup](docs/03-github-setup.md) | SSH keys, remotes, branches, syncing |
| 4 | [Deploys](docs/04-deploys.md) | GitHub Actions: push → SSH → reload |
| 5 | [Best practices](docs/05-best-practices.md) | Secrets, CLAUDE.md conventions, branch strategy |

