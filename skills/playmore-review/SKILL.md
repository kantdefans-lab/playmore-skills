---
name: playmore-review
description: AI review and classification of scored papers. Classifies each as ⭐必读 / 📖可读 / 💡了解 with a brief comment. Called by playmore after playmore-score.
argument-hint: ""
---

# Playmore Review

Takes the ranked paper list from playmore-score and classifies each paper.

## Classification Rules

### ⭐ 必读
- Score ≥ 7, OR
- SCI Q1/Q2 + keyword highly relevant, OR
- Directly addresses the search topic as a core contribution

### 📖 可读
- Score 4–6, OR
- Relevant methodology or dataset that may be useful
- Tangentially related but from a high-tier venue

### 💡 了解
- Score 2–3
- Background context, review papers, or loosely related work

## For each paper, output

```
{level} {title}
点评：{1–2 sentence comment explaining why this classification, what's notable}
```

Focus the comment on: what problem it solves, what's novel, why it matters for the search topic.

## Display summary

After classifying all papers, print:

```
共 {n} 篇 | ⭐ {x} 必读  📖 {y} 可读  💡 {z} 了解

⭐ 必读
1. ...

📖 可读
...

💡 了解
...
```

## Then

Pass the classified list to `/playmore-notes`.
