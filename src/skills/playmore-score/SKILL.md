---
name: playmore-score
description: Score and rank fetched papers using three dimensions: keyword relevance, journal tier, and SCI Q3+ boost. Personalizes weights using researcher profile. Called by playmore after playmore-fetch.
argument-hint: ""
---

# Playmore Score

Takes the unified paper list from playmore-fetch and scores each paper.

## Step 0 — Load keyword sources

Collect keywords from two sources, in priority order:

**Source A — Researcher profile** (higher priority)
Read `~/Desktop/playmore/profile.md` → keyword weight table.
Each keyword has a weight (1–10) reflecting how central it is to the user's research.

**Source B — Search keywords**
The keywords passed in $ARGUMENTS (the current search query).
These always get full weight regardless of profile.

**Merge rule:**
- If a keyword appears in both sources, use the profile weight
- If a keyword appears only in the search query, treat it as weight 5 (default)
- If a keyword appears only in the profile (weight ≥ 5), include it as a secondary scoring signal

---

## Scoring Dimensions

### 1. Keyword relevance (0–6 pts)

For each keyword, check title and abstract:

**Title match:**
- Profile weight 8–10: +3 pts
- Profile weight 5–7 or search keyword: +2 pts
- Profile weight 1–4: +1 pt

**Abstract match:**
- Any keyword: +1 pt (max 3 pts total from abstract)

**Cap:** keyword relevance score capped at 6 pts total.

**Negative keywords:** Read from profile.md → 负向关键词 section.
If any negative keyword appears in title or abstract → score = 0, exclude paper.

### 2. Journal tier (0–4 pts)

| Tier | Points |
|------|--------|
| SCI Q1 | 4 |
| SCI Q2 | 3 |
| SCI Q3 | 2 |
| SCI Q4 / CSSCI / 北大核心 | 1 |
| Unknown / preprint | 0 |

Determine tier by:
- Check `primary_location.source` from OpenAlex
- For Semantic Scholar: check `publicationVenue.type`
- For CNKI results: CSSCI = 1pt, 北大核心 = 1pt
- If journal name matches known Q1/Q2 venues, apply accordingly
- When uncertain, default to 0

**Profile journal boost:** If the journal appears in the user's 期刊偏好 list in profile.md, add +1 pt (max once per paper).

### 3. SCI Q3+ keyword boost (+2 pts)

If journal tier ≥ Q3 (score ≥ 2) AND keyword relevance > 0: add 2 bonus points.

---

## Final score

```
total = keyword_score + tier_score + journal_boost + q3_boost
```

Exclude papers with total < 2.

Sort descending by total score. Keep top 30.

---

## Output

Annotate each paper with:
```json
{
  "score": 7,
  "score_breakdown": {
    "keyword": 3,
    "tier": 2,
    "journal_boost": 0,
    "boost": 2
  },
  "journal_tier": "SCI Q2"
}
```

Pass ranked list to `/playmore-review`.
