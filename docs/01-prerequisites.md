# 01 — Prerequisites

Everything you need installed before touching Claude Code. Do these in order.

---

## You're ready

Proceed to [02 — Repo structure](02-repo-structure.md).

---

## 5. Claude Code

Claude Code is a CLI that also runs inside VS Code as an extension.

**Install the CLI:**
```bash
npm install -g @anthropic-ai/claude-code
```
*(Requires Node.js 18+. Install from [nodejs.org](https://nodejs.org/) if needed.)*

**Install the VS Code extension:**
Search "Claude Code" in the VS Code Extensions panel (Ctrl+Shift+X).

**Authenticate:**
```bash
claude
# Follow the browser prompt to connect your Anthropic account
```

---

## 4. GitHub CLI (`gh`)

The `gh` CLI lets you create repos, open PRs, and manage issues from the terminal.

**Windows:** Download the installer from [cli.github.com](https://cli.github.com/).

Authenticate:
```bash
gh auth login
# Choose: GitHub.com → SSH → your key → Login with a web browser
```

Verify:
```bash
gh auth status
# ✓ Logged in to github.com as yourname
```

---

## 3. GitHub account + SSH key

If you don't have a GitHub account: [github.com/signup](https://github.com/signup).

Generate an SSH key (run in Git Bash or terminal):
```bash
ssh-keygen -t ed25519 -C "your@email.com"
# Press Enter for all prompts to accept defaults
```

Add the public key to GitHub:
```bash
cat ~/.ssh/id_ed25519.pub
# Copy the output
```
GitHub → Settings → SSH and GPG keys → New SSH key → paste → Save.

Test it:
```bash
ssh -T git@github.com
# Hi username! You've successfully authenticated...
```

---

## 2. VS Code

Download from [code.visualstudio.com](https://code.visualstudio.com/).

Install these extensions (Ctrl+Shift+X → search):
- **GitLens** — inline blame, history, branch management
- **GitHub Pull Requests** — review PRs without leaving VS Code
- **Remote - SSH** *(optional)* — edit files directly on your server

---

## 1. Git

**Windows:** Download from [git-scm.com](https://git-scm.com/download/win). During install:
- Select "Git from the command line and also from 3rd-party software"
- Select "Use bundled OpenSSH" (avoids conflicts with Windows OpenSSH)
- Leave everything else default

Verify:
```bash
git --version
# git version 2.x.x
```
