# CLAUDE.md

Briefing for Claude Code. This repo is a setup guide — not a live system.
Do not attempt to deploy or SSH anywhere from here.

---

## House style: ordering & typography

These conventions apply across **every** A777ance repo — current and future. (Adopted 2026-06-05.)

- **Time-based content reads newest-first (reverse-chronological).** Logs, changelogs,
  decision logs (ADR / FIN), known-issues and issue trackers, FAQs, metrics and review
  logs, and "Handled For You" entries all lead with the most recent item. Apply this
  within the time-based *section* even when the whole file isn't time-based.
- **Alphabetical lists run Z → A** (descending).
- **Walkthroughs: reverse the blocks, keep the steps.** In a step-by-step guide, present
  the major sections/blocks in reverse order (last block first — it helps "block" the
  work), but keep the numbered steps *within* each block in forward order so every
  procedure stays followable. A walkthrough's table of contents mirrors the reversed
  block order. **Never renumber** — step and stage numbers stay fixed, so the intended
  execution order is always readable from the numbers.
- **Font: Gill Sans MT everywhere.** Every surface — customer-facing or internal — uses
  Gill Sans MT. Web/CSS stack:
  `'Gill Sans MT', 'Gill Sans', Calibri, 'Trebuchet MS', sans-serif`.

---

## Bifrost — active command schema (loads every session)

<!-- bifrost-briefing:start — GENERATED from localDNS/04-user-services/ai-orchestration/briefing-block.md by tools/sync-briefings.py. Do not hand-edit; edit the canonical file and re-run. -->

**Bifrost** is the A777ance command-composition schema — a keyboard-spatial notation
(`~ ! @ # $ % ^ & * ()` swept left→right, each glyph an *archetype* fulfilled by slash
commands + a plain-language sub-prompt). It is **active from the first token of every
session, in every repo:** adopt the `~` lazy-anchor posture — fire the first token ASAP
(the *model* stays high), let continuity coalesce mid-flight — and read Bifrost notation
per the schema whenever used.

- **Backbone:** `'` ignition (begins the Bifrost) · `~` continuity/lazy-anchor · `` ` ``
  descriptor (and, bare, the *expansion call*) · `!` cargo (a *manifest* — not executed on
  loading) · `@` source — **read-only** · `#` repo/destination — **write-allowed** · `$` sanity ·
  `%` compliance · `^` cars/lanes · `&` rotary — the **rabbit trail**, a nested Bifrost (also
  the sequential form) · `*` stop signal (red by default) · `()` governance (release
  conditions). Off-row `'`/`~`/`` ` `` stage; keys 1–4 **Preload** form a complete manifest —
  *what · from where · to where · against what*.
- **`@`/`#` are a permission pair, not a pair of arrows** (founder's rule, 2026-08-08). `@` is
  **read-only** — read it, never write it. `#` is **write-allowed** — what this run may create,
  modify or overwrite (still two-way). **They may overlap:** `@` alone = read-only, `#` alone =
  writable, both = read-write. Two slots, three states, one **mount table**. `@` still reads and
  `#` still writes, so every string already written stays valid — this only *adds* the guardrail,
  and gives the one-way door a question with an answer: *is every write inside `#`?*
- **`'` is always the signal to begin the Bifrost** (founder's rule, 2026-08-07 — fixes a
  mobile bug). Treat `'`, `’` (curly) and `′` as one glyph, and treat **presence and
  absence as the same string**: `' ~ !…` ≡ `~ !…`, `''` ≡ `'`. It marks *where* the Bifrost
  starts, never *what* runs — no sub-prompt, no `/how`, no intensity dial, `0` turbulence. A
  letter-flanked `'` (`don't`, `founder's`) is prose in a sub-prompt, not an ignition; only a
  free-standing `'` ignites. Never ask which apostrophe the phone chose.
- **A bare `'` (the whole message) = the reference call. Return this string and NOTHING else:**

  ```text
  ~!@#$%^&*()
  ```

  It is **the sweep itself** — exactly what sliding a finger down the row on a laptop puts on
  the screen. Not a legend, not a glossary, not a table: the row. So it is a **lookup, not a
  generation** — same bytes every call, every session, every model. No preamble, no trailing
  offer, no adaptation to the conversation. Answer *immediately*; it reads no file and fires no
  cargo. Glyph *meanings* live in the backbone above; the reference call hands back the
  **order**, which is the thing a phone cannot sweep for itself.
- **A bare descriptor — `` `…` `` with no backbone glyph in the message — is the *expansion
  call*.** The backticked text is a **seed**, and the answer is one complete, schema-compliant
  line with **every backbone slot filled in**, for the founder to read, parse and tweak:

  ```text
  ~ (fill in) ! (fill in) @ (fill in) # (fill in) $ (fill in) % (fill in) ^ (fill in) & (fill in) * (fill in) ( (fill in) )
  ```

  **The skeleton is the sweep, spaced** — strike the `(fill in)` slots and the whitespace and
  `~!@#$%^&*()` remains. `'` hands back the **order**; `` `seed` `` hands back the order **with
  the slots filled**. Echo the seed back on the `` ` `` line; fill **every** slot, never drop one
  (a complete draft is edited *down*); emit in Golden Rule order, so `K = 0` by construction;
  **`*` comes back RED, always** — an expansion is a *proposal*, nothing ran and no `#` was
  touched; and **collapse it** — where the surface renders HTML, ship it inside a `<details>`
  whose `<summary>` is the `~` requirement line. With a backbone glyph present, `` ` `` is the
  ordinary descriptor, unchanged. An empty descriptor returns the sweep. Unlike the bare `'`
  (a constant), an expansion **generates** — so the selector matters, and here it is the
  **human at the `*` gate**, not a vote.
- **`` ` `` and `&` are the same operation — nesting, at two positions** (founder's rule,
  2026-08-08). `&` is the **rabbit trail**: a digression you *come back from*, opening another
  full Bifrost inside this one. `` ` `` nests at staging, `&` nests on the road —
  `` `seed` `` ≡ `& seed` hoisted to position zero, which is why a bare descriptor can generate
  a line at all. So **expansion is recursive by construction**, and `&`'s "sequential" reading
  is just nesting seen from the parent's frame.
- **The greater traffic light is always the last bulwark** (founder's rule, 2026-08-08). Every
  nest **adds** a light; none removes one. An inner `*` going green releases its chunk **into
  its parent**, never into the world — only the outermost `*` stands between a `!` and an
  effect that cannot be recalled, however many inner gates already cleared. **Permissions
  intersect inward, gates conjoin outward:** a nested road may never write outside its parent's
  `#`, nor release past its parent's `*`. That is what lets `~` stay reckless at any depth —
  nesting multiplies the reasoning, never the exposure.
- **`*` cuts the road into Dispensations** — bounded, self-governing chunks. Governance has
  three outcomes: satisfied → green · **re-flagged** → return upstream via `&` (this is what
  lets a fixed string produce unbounded output) · unsatisfiable → eject to the shoulder.
- **The one-way door:** `~` rushes the reasoning, `*` gates the *effects* — anything
  irreversible (publish, deploy, send, push) rides past a light, which is exactly what makes
  the lazy start affordable.
- **Cars:** explicit `^` beats inferred. With no `^`, `!`'s command arity instantiates lanes
  1:1; with `^` present, `^` sets the lanes and `!`'s commands are the per-lane pipeline.
- **Guardrails survive a keyboard-mash:** `~` continuity, `$` sanity, `%` compliance — plus
  `*()` **governance**, the only one that repeats at every chunk boundary. `+` / repetition =
  more; `-` inverts into a stress test.

Canonical spec —
markdown: <https://github.com/a777ance/localDNS/blob/main/04-user-services/ai-orchestration/highway-notation.md>
· rendered page: <https://a777ance.github.io/localDNS/bifrost.html>

<!-- bifrost-briefing:end -->

---

## What this repo is

A step-by-step guide for setting up Claude Code in VS Code for homelab infra repos.
The living case study is [a777ance/localDNS](https://github.com/a777ance/localDNS),
which documents a self-hosted DNS + VPN + monitoring stack on an HP t630.

---

## Repo structure

```
claude-code-homelab/
├── CLAUDE.md               ← you are here
├── README.md               ← start here
├── docs/
│   ├── 01-prerequisites.md
│   ├── 02-repo-structure.md
│   ├── 03-github-setup.md
│   ├── 04-deploys.md
│   └── 05-best-practices.md
└── templates/
    ├── CLAUDE.md.template      ← copy this when starting a new infra repo
    ├── deploy.yml              ← GitHub Actions deploy workflow template
    └── .gitignore.template     ← secrets-safe gitignore for infra repos
```

---

## Working philosophy

- Docs here are meant to be followed sequentially by a new user
- All examples reference localDNS — link to the actual files, don't copy them
- When updating a step, verify it still matches localDNS's current state
- The goal is reproducibility: someone with a fresh Linux box should get to parity

---

## Related repos

- **[a777ance/localDNS](https://github.com/a777ance/localDNS)** — the case study; its CLAUDE.md links back here

---

## AI CTO state

Read `docs/ai-cto/context.md` in this repo for current open items.
The portfolio hub lives in `DESIGN-Full-Workflow-Integration-end-to-end-/docs/ai-cto/portfolio.md`.

---

## Branch policy — Yggdrasil and the Well of Mimir

<!-- branch-policy:start — GENERATED from localDNS/04-user-services/ai-orchestration/branch-policy-block.md by tools/sync-briefings.py. Do not hand-edit; edit the canonical file and re-run. -->

**`Yggdrasil` is the one standing working branch. Always push there, never to `main`.**
Founder's standing instruction (2026-08-08), superseding "push to `main`, no branches"
(2026-06-05).

- **One super-branch for the whole portfolio**, in every repo — no per-session branches.
  The branch-per-session habit is what produced 337 stale `claude/*` branches, 226 of them
  carrying commits that exist nowhere else.
- **`main` is the Well of Mimir** — vetted knowledge. It moves only by a pull request the
  founder approves. No cadence, no auto-merge: the Well fills when the founder decides it
  does. This is the Bifrost one-way door at portfolio scale — `main` is the outermost `*`,
  and no inner gate may release past it.
- **The spring is the founder, and it is out of scope for the machine.** An analog signal
  nothing here can sample or verify against. Yggdrasil and the Well are *channels*, not
  sources; every file in a repo is **transmission**, and transmission never promotes. A
  green check proves transcripts agree with **each other** — never that they agree with the
  founder. Only asking closes that gap.
- **Never overwrite doctrine.** Pull with `--ff-only` and nothing else — a fast-forward can
  only *add* commits, where a merge, rebase, or reset can silently rewrite founder-authored
  text. A session transcribes doctrine; it does not author it.
- **The tree is bigger than GitHub.** Yggdrasil spans the interacting systems — the t630
  stack, the LLM router, the NotebookLM bridge, Stripe, Setmore, the CRM — and GitHub is
  one root-well it drinks from.

**Push:** always `git push -u origin Yggdrasil`; retry with backoff on network failure.
Never `git push` to `main`, and never force-push either branch.

<!-- branch-policy:end -->
