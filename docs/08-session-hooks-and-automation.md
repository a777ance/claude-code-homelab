# 08 — Session Hooks and ADHD-Aware Automation

Lessons from the Chronikomicon project: automating progress tracking and reminders for a long-horizon creative goal (novel writing, 6-month deadline).

---

## The problem

Long projects stall not because of lack of ability, but because of friction at the start of each work session. For users with ADHD:

- Opening a project and not knowing where to pick up = session abandoned
- No visible progress = effort feels invisible = motivation collapse
- No external accountability = easy to skip "just today"

The solution is to eliminate all friction at session start and make progress visible without any manual action.

---

## SessionStart hook: automatic briefing

Claude Code runs `.claude/hooks/session-start.sh` at the start of every session. Use this to give the user immediate context — no terminal command required.

### Pattern: progress briefing hook

```bash
#!/bin/bash
# .claude/hooks/session-start.sh

REPO_DIR="${CLAUDE_PROJECT_DIR:-$(pwd)}"
TARGET=40000

# Count words in content files
WORD_COUNT=0
for f in "$REPO_DIR/manuscript/chapters"/*.md; do
  [ -f "$f" ] || continue
  [ "$(basename "$f")" = "TEMPLATE.md" ] && continue
  WORD_COUNT=$(( WORD_COUNT + $(wc -w < "$f") ))
done

# Days to deadline
DEADLINE_TS=$(date -d "2026-11-30" +%s)
DAYS_LEFT=$(( (DEADLINE_TS - $(date +%s)) / 86400 ))
WORDS_PER_DAY=$(( (TARGET - WORD_COUNT + DAYS_LEFT - 1) / DAYS_LEFT ))

# Display
printf '\n=== PROJECT STATUS ===\n'
printf '%d / %d words  |  %d days left  |  %d words/day needed\n' \
  "$WORD_COUNT" "$TARGET" "$DAYS_LEFT" "$WORDS_PER_DAY"
printf 'Minimum today: 220 words.\n\n'
```

### Register in `.claude/settings.json`

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash $CLAUDE_PROJECT_DIR/.claude/hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

**Always call via `bash`**, not directly. Files pushed via the GitHub API are created without the executable bit (`100644` not `100755`), so direct execution will fail silently or with a permission error.

---

## CLAUDE.md: session start protocol

The hook shows progress, but Claude's behavior at session start matters just as much. Add an explicit protocol to CLAUDE.md:

```markdown
## Session start protocol (every session)

1. State current word count, %, days to deadline, words/day needed
2. Ask one question: "What do you want to work on today?"
3. Stop. Wait for direction.

The user has ADHD. One clear prompt. No list of options. Overwhelming is a failure mode.
Minimum daily target: 220 words.
```

This pairs with the hook: the hook shows numbers, CLAUDE.md tells Claude what to *do* with them.

---

## GitHub Actions: monthly check-in issue

For long projects, create a scheduled Action that opens a GitHub Issue on the 1st of each month:

```yaml
# .github/workflows/monthly-checkin.yml
name: Monthly Check-in
on:
  schedule:
    - cron: '0 9 1 * *'   # 9am UTC, 1st of each month
  workflow_dispatch:       # allow manual trigger
jobs:
  checkin:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Count words
        id: stats
        run: |
          # ... word count logic ...
          echo "word_count=$WORD_COUNT" >> "$GITHUB_OUTPUT"
      - name: Create issue
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Monthly check-in: ${month} ${year}`,
              body: '...progress table + checklist...'
            });
```

This creates a persistent, closeable record of each monthly review — more durable than a reminder app.

---

## Automation stack summary

For a long-horizon creative project with ADHD constraints, the full stack is:

| Layer | Tool | Trigger | Purpose |
|-------|------|---------|--------|
| Session | `session-start.sh` + CLAUDE.md | Every Claude Code session | Instant context, no friction |
| Commit | `wordcount.yml` Action | Push to manuscript dir | Visible progress in CI |
| Monthly | `monthly-checkin.yml` Action | 1st of month (cron) | Accountability issue + checklist |
| Milestone | `git tag draft-1` (manual) | Writer decides | Named save points |

---

## Key lessons

1. **Visibility beats willpower.** Show the number. Make the gap obvious. Don't rely on the user to calculate their own progress.
2. **One prompt, not a menu.** ADHD users freeze when given options. The hook and CLAUDE.md together should produce exactly one actionable question at session start.
3. **The minimum matters more than the target.** 220 words/day (the floor) is more useful to surface than 40,000 words (the ceiling).
4. **Make the hook executable-safe.** Call `bash script.sh`, not `./script.sh`, when the file is created via API. The executable bit is a git attribute (`100755`) that must be set explicitly — the GitHub API defaults to `100644`.
5. **`workflow_dispatch` on every scheduled Action.** Always add a manual trigger so you can test the workflow without waiting for the cron date.
6. **Date arithmetic in bash.** On Linux (GNU coreutils): `date -d "YYYY-MM-DD" +%s`. On macOS (BSD): `date -jf "%Y-%m-%d" "YYYY-MM-DD" +%s`. Remote CI and Claude Code web sessions run Linux — use GNU form.
