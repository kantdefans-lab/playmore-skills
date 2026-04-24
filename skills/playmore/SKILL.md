---
name: playmore
description: Cross-platform academic paper tracker. Searches Google Scholar, CNKI, OpenAlex, and Semantic Scholar. Scores, reviews, and saves structured notes by concept. Use when the user wants to search, track, or review academic papers.
argument-hint: "[keywords] or 'schedule' to manage scheduled tasks"
---

# Playmore — Academic Paper Tracker

## Arguments

$ARGUMENTS contains either:
- Search keywords (e.g. `AI mental health`)
- `schedule` to manage scheduled tasks

## Routing

### If $ARGUMENTS is empty or contains keywords → search mode

Invoke sub-skills in order:
1. `/playmore-fetch $ARGUMENTS` — search across all sources
2. `/playmore-score` — score and rank results
3. `/playmore-review` — AI review and classification
4. `/playmore-notes` — generate and save notes

### If $ARGUMENTS is `schedule` → task management mode

Invoke `/playmore-schedule` to manage scheduled search tasks.

### If called with no arguments and scheduled tasks exist

Check `C:\Users\kantd\Desktop\playmore\schedule.json` for pending tasks.
If any task's date range includes today, run the full pipeline with that task's keywords.

## Notes storage root

`C:\Users\kantd\Desktop\playmore\`
