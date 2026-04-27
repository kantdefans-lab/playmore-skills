---
name: playmore-notes
description: Generate structured Markdown notes for reviewed papers and save them to concept-based folders. Called by playmore after playmore-review.
argument-hint: ""
---

# Playmore Notes

Generate one Markdown note per paper and save to `C:\Users\kantd\Desktop\playmore\`.

## Step 1 — Assign concept label

For each paper, identify its concept using title + abstract + keywords.

Concept format: `词×词` or `词×词×词` (e.g. `AI×心理健康`, `深度学习×情绪识别`, `LLM×教育`)

Rules:
- Be inclusive, not strict — if it roughly fits a concept, use it
- Check existing concept folders first: if a close match exists, reuse it
- Only create a new folder if no existing concept fits at all
- One paper can only belong to one concept folder (pick the most specific)

List existing concept folders:
```bash
ls "C:\Users\kantd\Desktop\playmore\"
```

## Step 2 — Create folder if needed

```bash
mkdir "C:\Users\kantd\Desktop\playmore\{concept}"
```

## Step 3 — Generate note content

Filename: `{year}_{first_author_lastname}_{slug_of_title}.md`

```markdown
---
title: {title}
authors: {authors}
year: {year}
journal: {journal}
journal_tier: {SCI Q1 / Q2 / Q3 / CSSCI / 北大核心 / unknown}
review: {⭐必读 / 📖可读 / 💡了解}
concept: {concept}
keywords: {keywords}
sources:
  - platform: {Google Scholar / CNKI / OpenAlex / Semantic Scholar}
    landing_url: {url}
    pdf_url: {pdf url or ""}
    doi: {doi or ""}
    arxiv_id: {arxiv id or ""}
score: {total score}
date_added: {today YYYY-MM-DD}
---

## 编辑评论

{full editorial comment from playmore-review, including:
- 总体评价
- 创新性和意义
- 技术质量
- 与相关工作的关系
- 应用前景和影响
- 主要问题
- 建议
- 决定}

## 摘要

{full abstract}
```

## Step 4 — Save the file

Write the note to:
`C:\Users\kantd\Desktop\playmore\{concept}\{filename}.md`

## Step 5 — Update index

After saving all notes, update `C:\Users\kantd\Desktop\playmore\INDEX.md`:

```markdown
# Playmore Index

_Last updated: {date}_

{for each concept folder, sorted alphabetically}
## {concept}
{for each note in folder, sorted by date_added desc}
- [{title}](./{concept}/{filename}.md) — {review emoji} {year} · {journal} · {journal_tier}
```

Write the full updated INDEX.md.
