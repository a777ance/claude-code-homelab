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

## The DIFF Command

At any point in a session — including deep inside write mode — the user may call DIFF. It is a hard-discrepancy audit between two representations of the same truth:

- **The repo** — canonical, ground truth, files as they exist
- **The bubble** — live session gist, accumulated through compacts, intentionally lossy, possibly drifted

### What DIFF hunts for

Hard discrepancies only — factual contradictions:
- Bubble says file is empty / repo file has content
- Bubble says deadline is December / repo says November
- Bubble has a character unnamed / map file has a name

**Not** discrepancies (ignore these):
- Tone differences
- Creative interpretation
- Emphasis shifts
- Anything that is intentional lossy artifact

### DIFF output format

```
DIFF — [session context]

[1] Bubble: ch01 is unwritten
    Repo:   ch01.md exists, 800 words
    → Vote: original / bubble

[2] Bubble: deadline is December
    Repo:   README says November 30
    → Vote: original / bubble
```

### The vote

- **Vote original** → repo is right. Bubble corrects in-session. No file write.
- **Vote bubble** → bubble is right. Repo is behind. Claude proposes the specific edit. User approves before any write.

### DIFF properties

- Does not compact
- Does not seal any zoom level
- Temporarily suspends write-mode lockout
- Can be called multiple times in one session
- Journal bubble resumes exactly where it left off after votes are applied

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
```

### 2. Pre-built zoom slices

Store pre-built T-1 and T-2 slices in a `context/` directory. Each slice:
- States its zoom level and purpose
- Lists exactly which files to read and follow
- Ends with a compact instruction and seal notice

```markdown
# T-1 Slice — [Topic]
*Read after T-0 compacted. Follow links. /compact. T-1 sealed.*

→ [file-1](../path/to/file1.md)
→ [file-2](../path/to/file2.md)

*After reading → /compact → write or zoom deeper.*
```

### 3. CLAUDE.md protocol encoding

Encode both the zoom sequence and DIFF command explicitly in CLAUDE.md. The model needs to see both protocols in T-0 to apply them correctly throughout the session.

---

## Key lessons

1. **The README is T-0, not documentation.** Rewrite it as a navigable map. Dense links, current state, rules — everything needed to orient in one read.
2. **Pre-build the slices.** Don't make the model decide what to read at T-1. Give it a file that specifies exactly which files to read. Reduces variance.
3. **Encode the seal in the file itself.** Each slice should end with the compact and seal instruction. The protocol is in the content, not just in CLAUDE.md.
4. **Lossy is a design choice.** For analytical tasks, lossy compression is a problem. For creative tasks, the distortion between map and gist is generative.
5. **DIFF is the safety valve.** It lets the user audit bubble drift without abandoning the journal bubble. Vote original or vote bubble — either is valid. The repo and the session can disagree, and the disagreement is resolved explicitly.
6. **The zoom can go negative infinitely.** T-2, T-3, T-4 — there is no floor. The protocol scales to any level of granularity without modification.
