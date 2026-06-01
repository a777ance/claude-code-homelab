# MANIFEST

Paged memory index for this repo. Claude reads this first, fetches only unbracketed files.

**Convention:**
- No brackets → active, fetch the file this session
- `[label]` → exists, read the summary only unless it becomes relevant

Full spec: `memory/manifest-convention.md`

---

| topic | file | summary |
|---|---|---|
| Manifest convention | memory/manifest-convention.md | Full spec for the MANIFEST + memory/ paged memory system used across all repos |
| Setup guide | memory/setup-guide.md | 5-step guide: prerequisites, repo structure, GitHub setup, deploy pipeline, best practices. Case study: localDNS on HP t630 |
| [reference] Templates | memory/templates.md | CLAUDE.md template, GitHub Actions deploy.yml, .gitignore for infra repos |
