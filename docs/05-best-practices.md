# 05 — Best Practices

Hard-won lessons from running Claude Code on a real homelab over time.

---

## You're done

Your repo is set up, automated, and battle-tested.

Come back to these docs when:
- You're onboarding another machine
- A deploy breaks and you're debugging the Action
- You want to add a new service to the stack

---

## VS Code + Claude Code tips

- **Use the inline diff view** before accepting changes. Cmd/Ctrl+click the file path Claude references to jump directly to the line.
- **Open the terminal panel** alongside Claude — you can run verification commands immediately after Claude makes a change.
- **Claude Code branches** show up in VS Code's Source Control panel. You can switch, diff, and merge without leaving the editor.
- **The `#` key in Claude Code** lets you reference a specific file directly in your prompt, which is faster than describing it.

---

## Keeping SETUP.md current

*(localDNS has since absorbed SETUP.md into its README.md — read "SETUP.md" below as "your
reproduction guide, wherever it lives." See the note in [02 — Repo structure](02-repo-structure.md).)*

Every time you make a change to the live system, ask: "If this machine died right now, could someone rebuild from SETUP.md?"

If the answer is no, update SETUP.md before landing the change.

A useful test: periodically read SETUP.md cold and try to find steps that are missing, out of order, or no longer accurate. The homelab changing is normal — documentation rot is the enemy.

---

## When Claude makes a change you didn't ask for

This happens. Claude may "clean up" adjacent code, add comments, or restructure a config when you only asked it to change one value.

Prevention:
- Be specific in your requests: "Change only the `cache-size-rrset` value in `unbound/tuning.conf` to 32m"
- Ask Claude to show you the diff before applying changes to sensitive files

Recovery:
```bash
git diff                    # see what changed
git checkout -- <file>      # revert a single file
git reset HEAD~1            # undo the last commit (keeps changes staged)
```

---

## Branch strategy in practice

Two workable models — pick one per repo and write it into that repo's CLAUDE.md:

**Solo / founder-direct (what localDNS uses today).** Push coherent, deployable commits
straight to `main` — no PRs, no parked feature branches. This is localDNS's standing
instruction (its CLAUDE.md §3): with one operator and a manually deployed box, a branch
is just a place for work to go stale. The discipline moves into the commit itself: every
commit must leave the setup guide able to rebuild the system.

**Branch-and-review (for teams, or once more than one person touches the repo).**
Claude Code auto-creates branches named `claude/adjective-noun-XXXXX`. Treat these as throwaway working branches:

1. Review the diff before merging to `main`
2. Merge with a squash commit so `main`'s history stays clean
3. Delete the `claude/` branch after merge

For your own changes:
- `feat/add-unbound-tuning` — feature work
- `fix/pihole-upstream` — bug fixes
- Direct commits to `main` only for documentation changes

---

## Secrets handling

**Never commit:**
- WireGuard private keys
- SSH private keys
- API tokens
- Passwords

**Do commit (with placeholders):**
- WireGuard public keys
- Service config files with `# PLACEHOLDER` where secrets go
- Docker Compose files with `${VAR}` syntax — the actual values live in `.env` files that are gitignored

**In CLAUDE.md, call out which values are placeholders:**
```markdown
## Known issues
| Issue | Action |
|-------|--------|
| `WEBPASSWORD` in pihole compose | Placeholder — do not commit real value |
```

---

## CLAUDE.md conventions

**Be specific, not generic.** "This is a homelab config repo" is useless. Tell Claude:
- What hardware you're running on
- What OS and kernel
- What NIC names to expect (`enp1s0`, not "the ethernet port")
- What Docker network mode each container uses and why

**Include a verification section.** A set of commands Claude can suggest to confirm the system is healthy after a change. Without this, Claude guesses.

**Keep it honest.** Include "Known issues" — things that are broken or in progress. Claude will try to fix things that aren't broken if it thinks they should be working.

**Don't put deployment instructions in CLAUDE.md.** Put those in your reproduction guide (SETUP.md, or README.md if you've merged them like localDNS). CLAUDE.md is operational context; the reproduction guide is what rebuilds the box.
