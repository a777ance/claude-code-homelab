# AI CTO Context — claude-code-homelab

Read alongside the portfolio hub: `../DESIGN-Full-Workflow-Integration-end-to-end-/docs/ai-cto/portfolio.md`.

**Last updated:** 2026-06-04

---

## What this repo is

A step-by-step guide for setting up Claude Code (VS Code + CLI) for homelab infrastructure repos. The case study throughout is `localDNS`. This repo is meta — it documents how to work, not what to build. Do not attempt to deploy or SSH anywhere from here.

## Current state

- Docs 01–07: complete (prerequisites → VS Code and creative projects)
- Templates: `CLAUDE.md.template`, `deploy.yml`, `.gitignore.template`
- Lessons from Chronikomicon project: added
- AI CTO architecture: not yet documented here (ADR-001 lives in DESIGN)

## Open items

| Item | Priority | Notes |
| ---- | -------- | ----- |
| Add AI CTO session protocol to `05-best-practices.md` | P2 | Reference `DESIGN/docs/ai-cto/` as the pattern |
| Consider adding `08-multi-repo-workflows.md` | P3 | Hub-and-spoke pattern; cross-repo context loading |
| Keep all examples in sync with localDNS current state | Ongoing | After every localDNS structural change, check docs here |

## Conventions

- All examples must link to actual `localDNS` files — do not copy/paste them inline
- A change here is only valid if `localDNS` still matches the example
- This repo has no live system; do not add deploy or SSH instructions
