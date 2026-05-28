# CLAUDE.md

Briefing for Claude Code. This repo is a setup guide — not a live system.
Do not attempt to deploy or SSH anywhere from here.

---

## What this repo is

A step-by-step guide for setting up Claude Code in VS Code for homelab infra repos.
The living case study is [a777ance/localDNS](https://github.com/a777ance/localDNS),
which documents a self-hosted DNS + VPN + monitoring stack on an HP t630.

---

## Repo structure

```
claude-code-homelab/
├── CLAUDE.md               ← you are here
├── README.md               ← start here
├── docs/
│   ├── 01-prerequisites.md
│   ├── 02-repo-structure.md
│   ├── 03-github-setup.md
│   ├── 04-deploys.md
│   └── 05-best-practices.md
└── templates/
    ├── CLAUDE.md.template      ← copy this when starting a new infra repo
    ├── deploy.yml              ← GitHub Actions deploy workflow template
    └── .gitignore.template     ← secrets-safe gitignore for infra repos
```

---

## Working philosophy

- Docs here are meant to be followed sequentially by a new user
- All examples reference localDNS — link to the actual files, don't copy them
- When updating a step, verify it still matches localDNS's current state
- The goal is reproducibility: someone with a fresh Linux box should get to parity

---

## Related repos

- **[a777ance/localDNS](https://github.com/a777ance/localDNS)** — the case study; its CLAUDE.md links back here
