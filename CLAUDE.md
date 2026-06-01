# CLAUDE.md

Briefing for Claude Code. This repo is a setup guide — not a live system.
Do not attempt to deploy or SSH anywhere from here.

---

## What this repo is

A step-by-step guide for setting up Claude Code in VS Code for homelab infra repos.
The living case study is [a777ance/localDNS](https://github.com/a777ance/localDNS),
which documents a self-hosted DNS + VPN + monitoring stack on an HP t630.

---

## Memory system

All repos in this organisation use the MANIFEST + `memory/` paged memory convention.
See `memory/manifest-convention.md` for the full spec.

Short version:
- Read `MANIFEST.md` first, every session
- Fetch only files whose topic has **no brackets**
- `[label]` topics exist and the summary is enough — skip the fetch unless it becomes relevant
- Never move files; change the bracket state in MANIFEST instead

---

## Repo structure

```
claude-code-homelab/
├── CLAUDE.md                        ← you are here
├── MANIFEST.md                      ← read this first
├── README.md
└── memory/
    ├── manifest-convention.md       ← full spec for the memory system
    ├── setup-guide.md               ← the actual guide content
    └── templates.md                 ← CLAUDE.md, deploy.yml, .gitignore templates
```

---

## Related repos

- **[a777ance/localDNS](https://github.com/a777ance/localDNS)** — the case study
- **[a777ance/azure-lab](https://github.com/a777ance/azure-lab)** — Azure equivalent of the homelab
