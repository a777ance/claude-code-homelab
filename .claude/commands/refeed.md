---
description: Lossless refeed — reload the latest on-disk briefing and this repo's canonical manifest after a /clear
allowed-tools: Bash(git fetch:*), Bash(git log:*), Bash(git status:*), Read
---

You have just been cleared. Re-establish the working briefing **from disk** so this
session is running on the *latest* docs, not a stale in-context copy. Do this fast
and losslessly — read the whole manifest, drop nothing, add nothing.

## 1. Prove you have the latest

Run these (read-only — never merge or check out here):

```bash
git fetch origin --quiet 2>/dev/null || true
git log -1 --format='local  %h  %ci  %s' -- CLAUDE.md
git log -1 --format='remote %h  %ci  %s' origin/HEAD -- CLAUDE.md 2>/dev/null || true
git status --short -- CLAUDE.md
```

If the **remote** copy is newer than **local**, stop and tell me — the on-disk copy
is behind and needs `git pull --ff-only` before the refeed means anything. If they
match (or there's no remote), the working tree is the latest — continue.

## 2. Read the lossless manifest (in one batch)

Read all of these from the working tree, in parallel:

1. `CLAUDE.md` — the briefing (this repo is a setup guide, not a live system — don't deploy from here).
2. `README.md` — start here.
3. `docs/ai-cto/context.md` — current open items (AI CTO state).

These **are** the lossless set for this repo. Anything else is pulled on demand,
not part of the standing briefing.

## 3. Confirm ready — one line

Report exactly: the `CLAUDE.md` revision you loaded (`<short-hash> <date>`), that
every manifest file was read, and any drift you noticed between them. Then stop and
wait for work. Do not summarize the manifest back to me — just confirm the feed is
fresh.
