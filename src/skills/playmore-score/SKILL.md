---
name: playmore-score
description: Score and rank fetched papers using three dimensions: keyword relevance, journal tier, and SCI Q3+ boost. Called by playmore after playmore-fetch.
argument-hint: ""
---

# Playmore Score

Takes the unified paper list from playmore-fetch and scores each paper.

## Scoring Dimensions

### 1. Keyword relevance (0–5 pts)

Read keywords from `C:\Users\kantd\Desktop\playmore\config.json` → `keywords`.

- Title contains keyword: +2 pts each (max 4)
- Abstract contains keyword: +1 pt each (max 3)
- Negative keyword match → score = 0, exclude paper

### 2. Journal tier (0–4 pts)

| Tier | Points |
|------|--------|
| SCI Q1 | 4 |
| SCI Q2 | 3 |
| SCI Q3 | 2 |
| SCI Q4 / CSSCI / 北大核心 | 1 |
| Unknown / preprint | 0 |

Determine tier by:
- Check `primary_location.source` from OpenAlex (has `is_in_doaj`, impact factor hints)
- For Semantic Scholar: check `publicationVenue.type`
- For CNKI results: CSSCI = 1pt, 北大核心 = 1pt
- If journal name contains known Q1/Q2 venues from memory, apply accordingly
- When uncertain, default to 0

### 3. SCI Q3+ keyword boost (+2 pts)

If journal tier ≥ Q3 (score ≥ 2) AND keyword relevance > 0: add 2 bonus points.

## Final score

```
total = keyword_score + tier_score + q3_boost
```

Exclude papers with total < 2.

Sort descending by total score. Keep top 30.

## Output

Annotate each paper with:
```json
{
  "score": 7,
  "score_breakdown": {
    "keyword": 3,
    "tier": 2,
    "boost": 2
  },
  "journal_tier": "SCI Q2"
}
```

Pass ranked list to `/playmore-review`.
