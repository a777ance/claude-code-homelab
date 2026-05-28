# 03 — GitHub Setup

Create the repo, connect your local folder, and set up a clean branching workflow.

---

## Create the repo

```bash
# With gh CLI (recommended):
gh repo create my-homelab --private --description "Homelab config snapshot" --clone

# Or create it at github.com/new, then:
git clone git@github.com:yourname/my-homelab.git
cd my-homelab
```

Use **private** for anything containing network topology, IP addresses, or service configs — even without secrets, this information is useful to attackers.

---

## Initial commit

```bash
# In your repo folder:
git add CLAUDE.md README.md SETUP.md
git commit -m "Initial commit: add CLAUDE.md and README"
git push -u origin main
```

Start with just the documentation. Add service configs one at a time so each commit is meaningful.

---

## Branch strategy

| Branch | Purpose |
|--------|---------|
| `main` | Always deployable. Every commit here should leave the system in a working state. |
| `claude/...` | Claude Code's working branches (auto-named) |
| `feat/...` | Your feature/change branches |

Never commit directly to `main` for non-trivial changes. Open a PR, even if you're the only one reviewing it — the diff view is useful.

---

## Syncing the repo to your server

**Option A: Pull on the server (simplest)**

SSH into your server and pull:
```bash
ssh user@192.168.1.118
cd ~/my-homelab
git pull
```
Then manually copy files to their system paths and reload services.

**Option B: GitHub Actions (automated — see [04-deploys.md](04-deploys.md))**

Push to `main` → GitHub Action SSHes into your server → copies files → reloads services.

---

## Keeping Claude Code's branches tidy

Claude Code creates branches named `claude/adjective-name-XXXXX`. These pile up. Clean them periodically:

```bash
# Delete merged claude/* branches locally
git branch | grep 'claude/' | xargs git branch -d

# Delete them on remote too
git push origin --delete $(git branch -r | grep 'origin/claude/' | sed 's/origin\///')
```

Or in GitHub: Settings → Branches → enable "Automatically delete head branches."

---

## SSH config for multiple servers

If you manage more than one machine, set up `~/.ssh/config`:

```
Host homelab
    HostName 192.168.1.118
    User youruser
    IdentityFile ~/.ssh/id_ed25519

Host homelab-vpn
    HostName 10.8.0.1
    User youruser
    IdentityFile ~/.ssh/id_ed25519
```

Then `ssh homelab` works from anywhere, including from GitHub Actions.

---

Proceed to [04 — Deploys](04-deploys.md).
