# 08 — Token Efficiency & Hybrid Routing

How to spend fewer tokens (and dollars) on the A777ance repos without losing quality —
and how to put the local LLM stack you already built (`localDNS/10-ai-orchestration`) to
work as the cheap tier. Current as of **June 2026**; this moves fast, so treat the dated
facts below as perishable and re-check the linked sources before a big change.

The one-line version: **match the model to the task, keep the constant baseline small,
let the cache do its job, and send the boring work to the box.**

---

## A. The biggest levers (ranked by impact, for *these* repos)

### 1. Trim the constant baseline — `CLAUDE.md` is loaded every turn, every session

Every `CLAUDE.md` loads *before* Claude reads your task — a 4,000-token file is 4,000
tokens you pay on every single turn, all session. The `DESIGN-…/CLAUDE.md` is the heaviest
in the portfolio (full funnel diagram, two AI-officer state blocks, eight-row tables). When
a session touches the portfolio hub, the cross-references can pull **several repos'
`CLAUDE.md` into one context** — easily 10–15k tokens of standing overhead before a word of
work.

What to do:
- **Cut each `CLAUDE.md` to the ~40 lines Claude needs *every* time.** Push the rest
  (full stage map, deploy-path tables, known-issues logs) into the files it already links
  to — Claude reads those on demand only when the task needs them. The deploy-path table is
  the classic example: vital when deploying, dead weight when editing prose.
- **Don't restate house style in seven files.** It's identical everywhere. Keep the full
  rule in one repo and a one-line pointer in the others.
- **Measure it:** `/context` shows what's eating the window. Run it at session start once in
  a while and look at the `CLAUDE.md` line.

### 2. Match the model to the task — stop paying Opus rates for janitorial work

Opus 4.8 is **$5 / $25 per Mtok** (in/out); Fast Mode is **$10 / $50**. Sonnet 4.6 and
Haiku 4.5 cost a fraction of that. A "watch the PR / summarize / did-anything-change"
routine does not need a frontier reasoner — run those on **Haiku or Sonnet** and reserve
Opus (and Fast Mode) for hard reasoning, architecture, and tricky diffs.

This applies directly to **scheduled routines**: a monitor that fires on a timer and usually
finds nothing should be the cheapest model that can spot the exception. Default the watchers
to Haiku; escalate to Opus only inside the run, only when something real turns up.

### 3. Use the local stack you already built as the cheap tier

You have what most people writing these guides don't: a working hybrid gateway —
**LiteLLM on `ai.home.lan:4040`** fronting local Ollama tiers (`local-fast` qwen2.5:3b,
`local-smart` 7b, `local-reason`, `local-embed`) with Claude as overflow, and a privacy gate
in the Odin supervisor. The published rule of thumb: **~60–70% of real requests are simple**
(classify, extract, format, summarize), and pushing those to a local model cuts AI cost
**8–10×** with no quality loss. The gap isn't capability — it's *defaulting* to local for the
boring work instead of reaching for the API out of habit.

Send to the box, not the API:
- **Embeddings / RAG indexing** — already wired (`local-embed`, `rag.py`). Never pay for these.
- **Summarizing logs, classifying/triaging, extracting fields, first-pass drafts** —
  `local-fast`/`local-smart` handle these. Let them fall over to `cloud-overflow` only when
  they genuinely can't.
- **Anything touching real customer data** (the `customers` repo) — local is also the
  *privacy* win, not just the cost win: the lookup never crosses the Bifröst.

Keep on the Claude API: multi-file reasoning, real code/diffs, the hardest statement-copy
judgement calls, anything where a wrong answer is expensive.

### 4. Never spend a token on something deterministic

`check-docs.py` validates every internal link in the `DESIGN-…` repo — it's a Python script,
not an LLM job. Same for formatters, linters, and the `make statement` render. Run the tool;
don't ask Claude to "check the links." Wire these into a SessionStart hook or CI so they're
free and automatic.

---

## B. Claude Code session hygiene

- **One task per session.** Stale context from a finished task is both expensive (you carry
  it every turn) and a distraction (Claude weighs it). Switching topics? Start fresh.
- **`/compact` long sessions** to fold the history into a summary while keeping the working
  mental model. **`/recap`** (new this spring) summarizes where you left off without
  replaying the whole conversation on resume.
- **Scope tight.** "Refactor the login function in `auth.ts`" pulls far less context than
  "refactor auth." Smaller scope → fewer file reads → fewer tokens → more focused output.
- **Reference files with `#`** instead of describing them — Claude reads the one file, not a
  search across the repo.
- **Prefer CLI over MCP** for bulk facts (`gh`, `git` are leaner than an MCP server, which
  adds a per-tool listing to context). Disable MCP servers you aren't using with `/mcp`.
- **Spend subagents deliberately.** Agent teams give each member its own context window —
  reportedly **~7× the tokens** of a single session when they run in plan mode. Worth it for
  genuine parallel fan-out; wasteful for a job one agent could do.

---

## C. Let prompt caching work for you

A cache hit costs **~10%** of normal input price; Anthropic's automatic caching slides the
breakpoint forward as a conversation grows, so multi-turn sessions can see **up to ~90%**
input savings. Two habits capture most of it:

- **Keep the stable stuff stable.** `CLAUDE.md` and standing instructions sit at the front of
  context — if you don't churn them mid-session, they stay cached. Editing `CLAUDE.md`
  mid-session busts the cache for everything after it.
- **Mind the TTL on scheduled routines.** The default cache window is ~5 minutes. A routine
  that fires every 20 minutes re-reads its whole context cold every time. So: keep routine
  prompts short, batch the work into fewer/larger runs rather than many tiny frequent ones,
  and run the watcher on a cheap model (see A.2). Batch API processing is another **−50%** if
  a job can tolerate latency.

---

## D. About the prompt that commissioned this doc

The request was, roughly: *"Locate inefficiencies in our process… reduce token use…
better prompting… leveraging other AI… hybrid local/Claude… ANYTHING that could help.
Search the web… keep up to date… check the news. Thanks!"*

It's a great brainstorm and a textbook example of the thing it's asking about. Why it's
token-expensive, and the fix:

| What makes it costly | Cheaper way to ask |
| --- | --- |
| **Unbounded scope** ("ANYTHING… anything you could possibly think of") forces wide, expensive exploration with no stopping rule. | Name the 2–3 areas you actually care about most this week. |
| **No target output** — Claude has to guess whether you want a paragraph, a repo doc, or a 5-page report. | "Write ≤2 pages to `claude-code-homelab/docs/`" — which is what this became. |
| **No scope of repos / no priority** | "Focus on the DESIGN + customers repos; ignore the stubs." |
| **Open web mandate** ("search the web… check the news") on every run | Fine once; for a *recurring* routine, say "only search if >30 days since last run" so the timer firings stay cheap. |
| Politeness ("Thanks!", "Perhaps also") | Harmless and cheap — keep it if it keeps you sane. The cost is in scope, not courtesy. |

A tighter version that would have cost a fraction:

> *Review our Claude Code usage across the DESIGN and customers repos for token waste.
> Give me the top 5 fixes ranked by impact, ≤2 pages, written to
> `claude-code-homelab/docs/`. Cite current (2026) sources. Skip the stub repos.*

Same answer, far less exploration, and a defined finish line. The general rule:
**scope + output format + stopping rule** is most of prompt efficiency.

> Meta-note: a scheduled routine that re-runs broad web research every firing is itself an
> inefficiency. Let this doc be the baseline and have the recurring run only *diff* against
> it — "what changed since June 2026?" — rather than re-derive the whole thing.

---

## E. Quick wins checklist

- [ ] Run `/context` on a real session; trim whatever `CLAUDE.md` lines dominate it.
- [ ] Move deploy-path tables / known-issues logs out of `CLAUDE.md` into linked files.
- [ ] De-duplicate the house-style block — full text in one repo, pointer in the rest.
- [ ] Set scheduled/watch routines to Haiku or Sonnet, not Opus/Fast.
- [ ] Make `local-fast`/`local-smart` the *default* for summarize/classify/extract/draft.
- [ ] Confirm embeddings + RAG never hit the API (they shouldn't — verify `local-embed`).
- [ ] Wire `check-docs.py` + linters into a hook/CI; never LLM a deterministic check.
- [ ] Don't edit `CLAUDE.md` mid-session; batch routine work to beat the ~5-min cache TTL.
- [ ] Ask with scope + output format + stopping rule.

---

## Sources (June 2026 — re-check before relying on a figure)

- [Manage costs effectively — Claude Code Docs](https://code.claude.com/docs/en/costs)
- [What's new — Claude Code Docs](https://code.claude.com/docs/en/whats-new)
- [Prompt caching — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Pricing — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/pricing)
- [How to Reduce Claude Code Token Usage: 8 Proven Methods (2026) — Agensi](https://www.agensi.io/learn/how-to-reduce-claude-code-token-usage)
- [Claude Code Token Optimization (2026 Guide) — Build to Launch](https://buildtolaunch.substack.com/p/claude-code-token-optimization)
- [Hybrid Cloud-Local LLM: The Complete Architecture Guide (2026) — SitePoint](https://www.sitepoint.com/hybrid-cloudlocal-llm-the-complete-architecture-guide-2026/)
- [Run Local AI Models with Claude Code to Cut Costs 10x — MindStudio](https://www.mindstudio.ai/blog/run-local-ai-models-with-claude-code-cut-costs)
- [Claude Code June 2026: New Features — SitePoint](https://www.sitepoint.com/claude-code-june-2026-10-new-features-devs-need-to-know/)
