# 08 — AI process efficiency (token cost + prompting + hybrid routing)

A review of the **process between the operator and the AI** across the A777ance repos:
where tokens (and money) leak, where prompting can be tightened, and how the existing
local-LLM stack can carry more of the load. Grounded in this guild's actual setup
(the LiteLLM router in `localDNS/10-ai-orchestration/`, the seven CLAUDE.md files, and
the scheduled-routine workflow), not generic advice.

> Research snapshot: **2026-06-19.** This field moves weekly — re-verify prices and
> feature names against the linked docs before acting on a number. Section
> "Changelog / what changed" at the bottom leads newest-first per house style.

---

## TL;DR — the ranked wins for *this* setup

1. **Fix the recurring-routine pattern first (biggest leak).** A vague, open-ended
   routine (like the one that generated this doc) re-does broad web research and
   reloads every repo's full CLAUDE.md *every run*, on Opus 4.8 — the most expensive
   model — and risks notifying with the same findings repeatedly. Scope it, give it an
   output contract, tell it to diff against last time, and drop it to a cheaper model.
   See §1.
2. **Right-size the model per task.** Routines, monitoring, digests, and doc edits do
   not need Opus 4.8. Sonnet 4.6 is ~5× cheaper on output and Haiku 4.5 cheaper still;
   reserve Opus for genuine hard reasoning. See §2.
3. **Turn on prompt caching for your own API calls** (the LiteLLM cloud tiers + the
   LangGraph supervisor). ~90% off the repeated prefix. Claude Code already caches its
   system prompt for you; your *app* calls probably don't yet. See §3.
4. **De-duplicate CLAUDE.md.** The ~25-line house-style block is copy-pasted verbatim
   into all 7 repos and the DESIGN brief is very long — every session pays to load it.
   Trim and link. See §4.
5. **Use the Batch API for non-interactive bulk work** (monthly statement generation at
   scale, NotebookLM-bridge syncs). 50% off, and it stacks with caching. See §2.
6. **Let the local box do more.** The router already defaults local-first; push routine
   classification/extraction/summarization there so the cloud bill is only the ~10% of
   work that needs a frontier model. See §5.

---

## 1. The routine/prompt layer — the largest and least-obvious leak

The prompt that commissioned this review is a good teaching example, so critique it
plainly (it asked us to):

**What it does well:** states the goal, invites breadth, and asks for current/web-checked
answers — fine for a *one-off* exploration.

**Why it is expensive as a recurring routine:**

- **No scope or stop condition.** "ANYTHING that could help… search the web… check the
  news" forces a wide fan-out *every run*. On a schedule that is the same expensive
  research on repeat.
- **No "diff against last time."** Without "only surface what's *new* since the last
  run," it re-reports the same findings and burns a notification each time — training the
  reader to ignore them.
- **No output contract.** No length, no format, no destination. The model guesses, and
  guesses long.
- **Runs on the heaviest model with maximum context.** Opus 4.8 [1m] + every loaded
  repo's CLAUDE.md, per run.
- **Two questions in one** (analyze the process *and* critique this prompt) — fine, but
  unscoped each compounds the above.

**A tighter rewrite** (drop in as the routine prompt; XML blocks because Claude parses
them more reliably than prose — see §6):

```xml
<task>Weekly: surface only NEW, material ways to cut our Claude/AI cost or improve
our prompting + hybrid-routing process since the last run.</task>

<context>Stack: LiteLLM router (localDNS/10-ai-orchestration), local-first Ollama tiers
on the t630, Anthropic cloud overflow. We already use a privacy-gated reasoning ladder.
Last run's findings live in claude-code-homelab/docs/08-ai-process-efficiency.md.</context>

<constraints>
- Check Anthropic's changelog + 2–3 reputable sources; skip anything already in doc 08.
- If nothing materially new, send NO notification and exit. Silence is the success case.
- Budget your own effort: web search only, no repo-wide reads unless a finding needs it.
</constraints>

<output>If something new: a 5-bullet digest (each <20 words) + one concrete next action,
appended newest-first to doc 08. Notify with the single most important line.</output>
```

This swaps "do everything, every week" for "tell me only what changed, cheaply, or stay
quiet" — which is what a watch-routine is actually for.

**Cross-cutting routine hygiene** (applies to *every* scheduled routine, not just this one):

- **Make silence the default.** State explicitly when *not* to notify. A routine that
  pings "all clear" every run is negative value.
- **Give every routine a destination + a memory.** Append to a dated log it reads next
  time, so it can diff instead of re-deriving.
- **Pick the cheapest model that clears the bar** (next section).
- **Keep loaded-repo scope minimal.** Each repo added to a session loads its CLAUDE.md
  into context every turn. Only load the repos a routine actually touches.

---

## 2. Model right-sizing (and Batch)

Current `config.yaml` defaults `cloud-overflow`, `cloud-explore`, and `cloud-vision` to
**`anthropic/claude-opus-4-8`**. Opus is the right tool for the hardest reasoning and
wide synthesis — and the wrong tool for overflow, monitoring, extraction, and routine
edits, which are most of the volume.

| Work | Use | Why |
| ---- | --- | --- |
| Hard reasoning, architecture, cross-repo synthesis | **Opus 4.8** | Worth the premium for the ~10% that needs it |
| Code, diffs, structured build (already `cloud-code`) | **Sonnet 4.6** | Speed/cost sweet spot (~5× cheaper output than Opus) |
| Overflow, classify, extract, summarize, format | **Haiku 4.5** | Cheapest; handles the bulk |
| Non-interactive bulk (statements at scale, bridge syncs) | **Batch API** | ~50% off; stacks with caching |

Concrete change: point `cloud-overflow` at `claude-haiku-4-5` (or `sonnet-4-6`) instead
of `opus-4-8`, and let the LangGraph supervisor escalate to Opus only on an explicit
"hard" signal. Overflow is a *failover*, not the place to spend the most per token.

Also note (2026): the **1M context window no longer carries a price premium** for Opus
4.6+/Sonnet 4.6 — selecting 1M is free; you still pay per token used, so big context is
"use when needed," not "leave on."

---

## 3. Prompt caching — ~90% off the repeated prefix

Caching bills the stable prefix (system prompt, schema, long instructions) at ~10% on a
cache hit. Rules of thumb from current guidance: worth it at **3+ reads** within the
5-minute window, **5+ reads** for the 1-hour cache. Reported production savings land
**60–90%** on the prefix; combined with Batch, up to ~95% on eligible workloads.

- **Claude Code already does this** for its system prompt + CLAUDE.md — one reason a long
  CLAUDE.md is cheaper than it looks *within* a session (but not across cold starts, and
  not free — see §4).
- **Your own API calls likely do not.** LiteLLM passes `cache_control` through to
  Anthropic — mark the LangGraph supervisor's static system/instruction block and any
  large RAG context as cacheable.
- **The #1 caching bug:** a live timestamp in the cached prefix
  (`"Current time: 2026-06-19T14:32:15Z"`) invalidates the cache *every call*. Truncate
  to the day or move it out of the prefix.

---

## 4. CLAUDE.md de-duplication

- The **house-style block (~25 lines) is duplicated verbatim across all 7 repos.** It
  loads into context on every session for every repo. Keep one canonical copy (e.g. in
  this repo or DESIGN) and have each CLAUDE.md link to it with a 2-line summary, rather
  than inline the whole thing.
- The **DESIGN CLAUDE.md is very long.** A CLAUDE.md is read top-to-bottom every session;
  treat it like a cached system prompt — lead with the rules that change behavior, push
  reference detail to README/linked files. Current guidance: structure beats length;
  reasoning quality degrades past a few thousand tokens of instruction.
- **Tone:** newer Claude models respond *worse* to shouted emphasis. The CLAUDE.md files
  lean hard on `IMPORTANT`/`NEVER`/`MUST`/caps. Keep the few that are genuine guardrails
  (secrets, push-to-main); state the rest calmly. "Say it once, plainly" outperforms
  "say it in caps three times" — and costs fewer tokens.

---

## 5. Hybrid local + cloud — you are most of the way there

The router is already a textbook hybrid: a privacy gate pins sensitive work local, a
reasoning ladder (light distill local → full R1 on a rented GPU → cloud overflow),
local embeddings for RAG, graceful failover. That matches the 2026 reference
architecture (LiteLLM gateway + Ollama local + Claude cloud tier). Remaining gains are
incremental, not structural:

- **Move more *routine* volume off the cloud tier.** Typical workloads are ~60–70% simple
  (classify/extract/format) — those should resolve on `local-fast`/`local-smart`, with
  cloud reserved for the hard ~10%. Audit which tasks currently default to a `cloud-*`
  tier and demote the simple ones.
- **The economics check out:** a $60–100/mo cloud spend offsets a ~$489 local GPU in
  5–8 months — but only buy hardware once local *demand* is real; the t630 CPU tiers + a
  rented on-demand GPU already cover today's load without capex.
- **Keep the cloud tier cheap by default** (§2): overflow → Haiku/Sonnet, Opus on demand.
- **Watch for over-routing.** A hybrid layer adds a hop and a failure mode; only route
  where the savings beat the added latency/complexity. The current local-first default is
  the right bias.

---

## 6. Prompting mechanics that cut tokens (2026 consensus)

- **XML tags over markdown/numbered lists** for structuring Claude prompts:
  `<instructions> <context> <task> <output>`. Claude parses them more reliably and they
  compress intent.
- **Specify output length concretely:** "5 bullets, each under 15 words" beats "be
  concise." Unbounded output is where tokens quietly pile up.
- **Smaller, sequential prompts** beat one mega-prompt; reasoning degrades past ~3k
  tokens of instruction (sweet spot ~150–300 words of actual instruction).
- **Calm, direct phrasing** — drop `CRITICAL!`/`YOU MUST`; they hurt newer models.
- **Delegate heavy reading to subagents.** Anything that needs reading >3–4 large files
  is a subagent candidate: it explores in its own context and returns a summary, so the
  main session stays small and cheap. (This review used that pattern.)
- **Mind the real cost driver:** in agentic sessions the bill is dominated by accumulated
  context — session history, re-read files, tool outputs — not the prompt you typed. Keep
  sessions focused, start fresh for unrelated tasks, and lean on compaction.

---

## Changelog / what changed (newest first)

- **2026-06-19** — Claude Code **2.1.183** shipped today: adds `attribution.sessionUrl`
  to omit the claude.ai session link from commits/PRs in web + Remote Control sessions
  (relevant to our commit footer), blocks destructive git/IaC commands in auto mode
  unless asked, and prompts before writing code-executing files even in acceptEdits.
- **2026-06** — Claude Code rate limits doubled; Opus API limits raised. Auto mode now on
  Bedrock/Vertex/Foundry for Opus 4.7/4.8.
- **2026-03** — 1M context premium **removed** for Opus 4.6+/Sonnet 4.6; context
  compaction available via beta header `compact-2026-01-12` (reported ~15% fewer
  compaction events since 1M shipped).

---

## Sources

- [Claude Code changelog](https://code.claude.com/docs/en/changelog) · [Anthropic release notes — June 2026 (Releasebot)](https://releasebot.io/updates/anthropic/claude-code)
- [Prompt caching — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Prompt Caching: cut your bill 60% (AI Magicx)](https://www.aimagicx.com/blog/prompt-caching-claude-api-cost-optimization-2026)
- [Context windows — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows) · [Compaction — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/compaction) · [Context engineering (Claude Cookbook)](https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools)
- [Anthropic API pricing 2026 (Finout)](https://www.finout.io/blog/anthropic-api-pricing) · [Claude API pricing 2026 (MetaCTO)](https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration)
- [7 ways to reduce Claude Code token usage (KDnuggets)](https://www.kdnuggets.com/7-practical-ways-to-reduce-claude-code-token-usage) · [Claude Code token optimization (BuildToLaunch)](https://buildtolaunch.substack.com/p/claude-code-token-optimization) · [Claude Code advanced best practices 2026 (SmartScope)](https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/)
- [Hybrid cloud-local LLM architecture guide 2026 (SitePoint)](https://www.sitepoint.com/hybrid-cloudlocal-llm-the-complete-architecture-guide-2026/) · [Hybrid cloud-local AI cost optimization (BuildMVPFast)](https://www.buildmvpfast.com/blog/hybrid-cloud-local-ai-workflow-cost-optimization-2026) · [LLM model routing 2026 (Digital Applied)](https://www.digitalapplied.com/blog/llm-model-routing-2026-cost-quality-optimization-engineering-guide)
- [Best practices for prompt engineering (Claude)](https://claude.com/blog/best-practices-for-prompt-engineering) · [Prompt engineering best practices 2026 (Prompt Builder)](https://promptbuilder.cc/blog/prompt-engineering-best-practices-2026)
