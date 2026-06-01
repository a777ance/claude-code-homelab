# Templates

## CLAUDE.md template

Copy this when starting a new infra repo. Customise the stack section.

```markdown
# CLAUDE.md

Briefing for Claude Code.

## Memory system
Read MANIFEST.md first. Fetch unbracketed files. Bracketed topics exist — summary only.

## What this repo is
[one paragraph describing the service/stack]

## Stack
- OS: Ubuntu 24.04
- Services: [list]
- Deploy: push to main → GitHub Actions → SSH → reload

## Key files
- [path] — [what it does]

## Do not
- Commit secrets
- Push directly to main
- Restart services unnecessarily
```

---

## deploy.yml (GitHub Actions)

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/your-service
            git pull origin main
            systemctl reload your-service
            # or: docker compose up -d
```

Secrets to add in GitHub repo settings:
- `SSH_HOST` — server IP or hostname
- `SSH_USER` — deploy user (ideally a limited deploy user, not root)
- `SSH_PRIVATE_KEY` — private key whose public key is in `~/.ssh/authorized_keys` on the server

---

## .gitignore template

```gitignore
# Secrets — never commit these
*.env
.env*
*_secret*
*_private*
wireguard/privatekey
wireguard/*/privatekey

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
*.swp
```
