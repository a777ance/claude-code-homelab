# MANIFEST + memory/ Convention

Standard paged memory system for all repos in this organisation.
Designed to reduce token cost when Claude reads a repo.

---

## Problem it solves

Reading an entire repo each session wastes tokens on content that isn't relevant to the current task.
But not reading anything means Claude loses context on what exists.

---

## Structure

Every repo has exactly three things at the root:

```
README.md       ← human-facing intro, links to MANIFEST
MANIFEST.md     ← Claude reads this first, every session
memory/         ← all content files live here permanently
```

---

## MANIFEST format

```markdown
| topic | file | summary |
|---|---|---|
| Topic name | memory/filename.md | One sentence describing what is in the file |
| [label] Topic name | memory/filename.md | One sentence — exists but don't fetch |
```

### Bracket convention

| State | Meaning | Claude behaviour |
|---|---|---|
| No brackets | Active | Fetch the file and read it |
| `[label]` | Switched off | The summary is enough — skip the fetch |

The label is a signal, not just a toggle. Examples:

| Label | Meaning |
|---|---|
| `[reference]` | Stable background info, rarely changes |
| `[draft]` | In progress, not ready to act on |
| `[blocked]` | Waiting on something external |
| `[archived]` | Done, kept for record only |
| `[Judas]` | Deprioritised for now |

Any label works — the important thing is that *something* in brackets signals "don't fetch".

---

## Rules

1. **Files never move.** Paths in `memory/` are permanent. Changing bracket state in MANIFEST is the only operation needed to promote or demote a topic.
2. **One file per topic.** Keep files focused. If a topic grows too large, split it and add both to MANIFEST.
3. **Summaries must be self-contained.** The summary column should give Claude enough signal to decide whether to fetch — a vague summary defeats the purpose.
4. **README stays thin.** README is for humans. MANIFEST is for Claude.
5. **CLAUDE.md describes the system.** Every repo's CLAUDE.md should mention the memory system and instruct Claude to read MANIFEST first.

---

## Workflow

### Starting a session

1. Claude reads `MANIFEST.md`
2. Fetches files for all unbracketed topics
3. Has summaries for all bracketed topics
4. Proceeds with the task

### Promoting a topic (bracketed → active)

Edit one cell in MANIFEST — remove the brackets from the topic name.
No files move. The path is unchanged.

### Demoting a topic (active → bracketed)

Edit one cell in MANIFEST — add `[label]` to the topic name.
No files move.

### Adding a new topic

1. Create `memory/new-topic.md`
2. Add a row to MANIFEST with a good summary
3. Start unbracketed if it's immediately relevant, bracketed if not

---

## Why not folders (accessible/archived)?

Folder-based approaches require moving files when promoting/demoting, which:
- Creates git history noise (commits just for memory management)
- Breaks any references to the old path
- Requires more operations than editing one cell

The MANIFEST approach changes nothing on disk — only the index changes.
