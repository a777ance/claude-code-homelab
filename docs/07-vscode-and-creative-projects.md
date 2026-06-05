# 07 — VS Code for Creative Projects

Lessons from using VS Code + Claude Code + Git as a full creative production stack, replacing Word processors, outliners, and editing software.

---

## Checklist: setting up a new creative project repo

- [ ] `CLAUDE.md` with terminology, layer table, working rules
- [ ] `.vscode/settings.json` with prose-optimized settings
- [ ] `.vscode/extensions.json` with writing extensions
- [ ] `.vscode/tasks.json` with local build task
- [ ] `.github/workflows/build.yml` for automated manuscript builds
- [ ] `principles/` directory with a README
- [ ] `manuscript/chapters/` directory ready for chapter files
- [ ] `workflow/drafts.md` documenting the draft/tagging system
- [ ] `DRAFTS.md` at root as a living milestone log
- [ ] `.gitignore` excluding `build/` and editor temp files

## Layer separation for creative projects

Creative projects benefit from the same layer discipline as infra projects:

```
principles/     ← theory, structure, organizing ideas (not the work)
manuscript/     ← the actual output (the work)
reference/      ← source material (immutable once committed)
workflow/       ← process docs (how you work)
```

Theory and output are kept strictly separate. Principles inform the work but never appear inside it. Reference material is never modified.

---

## Build pipeline for prose

Use pandoc + GitHub Actions to build PDF/epub/HTML on every push to `main`:

```yaml
# .github/workflows/build.yml
on:
  push:
    branches: [main]
    paths: ['manuscript/**']
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt-get install -y pandoc
      - run: |
          mkdir -p build
          cat manuscript/chapters/*.md | pandoc - -o build/output.pdf
      - uses: actions/upload-artifact@v4
        with:
          name: manuscript
          path: build/
```

Local build shortcut: `Ctrl+Shift+B` in VS Code (configure in `.vscode/tasks.json`).

---

## Git for draft management

### Three levels of version control for creative work

**Commits** — granular save points. Commit after every meaningful session.
```bash
git commit -m "ch03: rough draft of the midnight dispensation"
git commit -m "ch03: cut 400 words, tightened the inn scene"
```

**Tags** — named milestones. Create when a full draft is complete.
```bash
git tag draft-1 -m "First complete draft, ~18k words"
git push origin draft-1
```

**Branches** — parallel experiments. Use when trying something that might not work.
```bash
git checkout -b experiment/ch06-second-person
# Try it. If it works, merge. If not, leave the branch — never delete.
```

### Comparing drafts

```bash
# What changed between draft 1 and draft 2 in chapter 3?
git diff draft-1 draft-2 -- manuscript/chapters/03-*.md

# Read an old draft
git checkout draft-1        # read it
git checkout main           # return

# Recover a deleted passage
git log --oneline manuscript/chapters/05-*.md
git show <sha>:manuscript/chapters/05-title.md
```

### Never delete experiment branches

Abandoned experiments are still part of the record of the work. Leave them. They cost nothing.

---

## Essential extensions for writing

```json
// .vscode/extensions.json
{
  "recommendations": [
    "streetsidesoftware.code-spell-checker",
    "yzhang.markdown-all-in-one",
    "shd101wyy.markdown-preview-enhanced",
    "ms-vscode.wordcount",
    "stkb.rewrap",
    "eamodio.gitlens",
    "anthropic.claude-code"
  ]
}
```

When a collaborator opens the repo, VS Code prompts them to install all of these automatically.

---

## VS Code settings for prose

A coding-optimized VS Code setup is wrong for writing. Override it per-project with `.vscode/settings.json`:

```json
{
  "editor.wordWrap": "bounded",
  "editor.wordWrapColumn": 80,
  "editor.lineNumbers": "off",
  "editor.minimap.enabled": false,
  "editor.fontSize": 16,
  "editor.lineHeight": 30,
  "editor.padding.top": 24,
  "editor.glyphMargin": false,
  "editor.folding": false,
  "workbench.activityBar.location": "hidden",
  "breadcrumbs.enabled": false,
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000,
  "[markdown]": {
    "editor.quickSuggestions": { "other": false }
  }
}
```

Key changes: word wrap on, line numbers off, minimap off, generous line height, auto-save, no code suggestions in markdown.

---

## The core idea

A Git repo + VS Code + Claude Code can replace every creative writing tool:

| Replaced tool | Git/VS Code equivalent |
|---------------|------------------------|
| Word processor | VS Code + Markdown |
| Outliner (Scrivener, etc.) | `principles/` layer in the repo |
| Version history / Track Changes | Git commits and diffs |
| Named drafts ("final_v3.docx") | Git tags (`draft-1`, `draft-2`) |
| Backup system | Remote GitHub repo |
| Style/grammar checker | Code Spell Checker + Vale extension |
| Build/export to PDF/epub | Pandoc via GitHub Actions or local task |

The repo is the single source of truth. Nothing lives outside it.

---
