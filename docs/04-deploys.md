# 04 — Deploys

Automate: push to `main` → files land on your server → services reload.

Uses GitHub Actions + SSH. No agent or daemon needed on the server.

---

## How it works

```
git push → GitHub Actions runner → SSH into server → rsync files → reload services
```

The runner SSHes into your homelab server using a deploy key stored as a GitHub secret.
It copies changed files, then runs the appropriate reload commands.

---

## Step 1: Create a deploy key

On your server, generate a key dedicated to deploys (no passphrase):
```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/deploy_key -N ""
```

Add the **public key** to your server's authorized_keys:
```bash
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys
```

Add the **private key** as a GitHub secret:
- Go to your repo → Settings → Secrets and variables → Actions → New repository secret
- Name: `DEPLOY_KEY`
- Value: contents of `~/.ssh/deploy_key` (the private key, no `.pub`)

Also add these secrets:
- `DEPLOY_HOST` — your server's IP, e.g. `192.168.1.118`
- `DEPLOY_USER` — your SSH username, e.g. `dpall`

---

## Step 2: Add the workflow

Create `.github/workflows/deploy.yml` in your repo.
A ready-to-use template is at [templates/deploy.yml](../templates/deploy.yml).

The workflow:
1. Triggers on push to `main`
2. Sets up SSH with your deploy key
3. rsyncs the repo to the server
4. Runs reload commands for any services whose files changed

---

## Step 3: Scope what gets deployed

Don't deploy everything blindly. Your workflow should:
- Copy only the service config files (not README, CLAUDE.md, docs/)
- Run reload commands only for services whose files actually changed
- Fail loudly if SSH can't connect (don't silently skip)

---

## Reload commands by service type

| Service | Reload command |
|---------|---------------|
| systemd unit | `sudo systemctl daemon-reload && sudo systemctl restart <service>` |
| Docker Compose | `cd ~/<folder> && docker compose up -d` |
| unbound | `sudo systemctl restart unbound` |
| UFW | `sudo bash ufw/setup.sh` |
| WireGuard | `sudo systemctl restart wg-quick@wg0` |

---

## Restricting sudo for the deploy user

Don't give the deploy SSH key full sudo. Create a sudoers file that only allows the specific commands it needs:

```bash
sudo visudo -f /etc/sudoers.d/github-deploy
```

```
# /etc/sudoers.d/github-deploy
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart unbound
deploy ALL=(ALL) NOPASSWD: /bin/systemctl daemon-reload
deploy ALL=(ALL) NOPASSWD: /usr/bin/docker compose *
```

This means a compromised deploy key can only restart specific services — not become root.

---

## Testing without pushing

```bash
# Dry-run rsync to see what would change:
rsync -avzn --delete ./unbound/ user@192.168.1.118:/etc/unbound/unbound.conf.d/

# Test SSH connection the Actions runner will use:
ssh -i ~/.ssh/deploy_key user@192.168.1.118 "echo ok"
```

---

Proceed to [05 — Best practices](05-best-practices.md).
