# 09 — The T-Minus Zoom Protocol

A context management architecture for long-horizon creative or research projects where token dilution degrades output quality as the session grows.

Originated in the Chronikomicon novel project.

---

## The problem

In large repos, reading many files before writing dilutes the model's attention. Token quality degrades as more content competes for the same working memory. Naive solutions (read less, summarize manually) either lose important context or require human effort.

The deeper issue: there is no standard protocol for *how much* context to load, in *what order*, for *what purpose*.

---

## The T-Minus model

Zoom levels are numbered T-0, T-1, T-2, T-N. T-0 is root (widest). Each subsequent level is a tighter focus. There is no floor — zoom can go infinitely deep.

```
T-0   Full repo map (README nerve center)           →  /compact  →  T-0 sealed
T-1   Topic slice (e.g. chapter structure)          →  /compact  →  T-1 sealed
T-2   Scene slice (e.g. one chapter + its links)    →  /compact  →  T-2 sealed
T-N   Any finer focus needed                        →  /compact  →  T-N sealed
WRITE Repo closed. Journal bubble only. No reads.   Output here.
```

**Perimeter rule:** Once a level is compacted, it is sealed. No file from that level or above may be re-read for the remainder of the session. The bubble shrinks with each compact.

**Lossy is intentional:** The gist that survives each compact is not a perfect summary. This is a feature, not a bug. For creative work, the compression introduces variation and forces interpretation rather than verbatim recall — the writer works with the distortion.

---

## Implementation

### 1. T-0: The nerve center README

The root README is not a project description — it is a **hyperlinked map** of the entire repo. Dense. Cross-linked. Designed to be read once and compacted.

```markdown
# PROJECT NAME

*This file is T-0. Read first every session. Follow links. Then /compact.*

## Map
- [Organizing principle](map/principles/01-name.md)
- [Characters](map/characters/README.md)
- [Chapter index](manuscript/chapters/)
...

## Rules
...
```

### 2. Pre-built zoom slices

Store pre-built T-1 and T-2 slices in a `context/` directory. Each slice:
- States its zoom level and purpose
- Lists exactly which files to read
- Includes the links to follow within those files
- Ends with a compact instruction and seal notice

```markdown
# T-1 Slice — [Topic]
*Read after T-0 compacted. Follow links. /compact. T-1 sealed.*

→ [file-1](../path/to/file1.md)
→ [file-2](../path/to/file2.md)

*After reading → /compact → write or zoom deeper.*
```

### 3. CLAUDE.md protocol

Encode the zoom sequence and perimeter rule explicitly in CLAUDE.md:

```markdown
## T-Minus Zoom Protocol

| Level | Read | Action | Result |
|-------|------|--------|--------|
| T-0 | README.md | /compact | T-0 sealed |
| T-1 | context/t1-[topic].md | /compact | T-1 sealed |
| T-N | zoom as needed | /compact each time | each level seals |
| WRITE | repo closed | conversation only | journal bubble |

Once a level is sealed, do not re-read its files. Lossy gist is intentional.
```

### 4. Hyperlink discipline

The protocol only works if files are cross-linked. Every file should link to:
- Its parent index
- Related sibling nodes
- Downstream dependencies

Claude follows links *within* a zoom level before compacting. This ensures the gist captures the graph, not just the root file.

---

## When to use this pattern

Use T-minus zoom when:
- The repo has more content than fits in a useful working context
- Output quality degrades as session length grows
- The work requires deep focus (writing, design, analysis) not wide exploration
- You want the model to be *informed* by the full repo without being *diluted* by it

Do not use it when:
- The task genuinely requires holding all content simultaneously (e.g. find-and-replace across files)
- The repo is small enough to read fully without quality loss

---

## Key lessons

1. **The README is T-0, not documentation.** Rewrite it as a navigable map, not a project description. Dense links, current state, rules — everything Claude needs to orient in one read.
2. **Pre-build the slices.** Don't make Claude decide what to read at T-1. Give it a file that tells it exactly which files to read and follow. Reduces cognitive overhead and session variance.
3. **Encode the seal in the file itself.** Each slice should end with "After reading → /compact → T-N sealed." The instruction is in the content, not just in CLAUDE.md.
4. **Lossy is a design choice.** For analytical tasks, lossy compression is a problem. For creative tasks, the distortion between the map and the gist is generative. Design for your domain.
5. **The zoom can go negative infinitely.** T-2, T-3, T-4 — there is no floor. The protocol scales to any level of granularity without modification.
