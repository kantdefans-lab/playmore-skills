---
name: playmore-notes
description: Generate structured Markdown notes for reviewed papers and save them to concept-based folders. Updates INDEX.md. Called by playmore after playmore-review.
argument-hint: ""
---

# Playmore Notes

Generate one Markdown note per paper and save to `~/Desktop/playmore/`.

## Step 1 — Assign concept label

For each paper, identify its concept using title + abstract + keywords.

Concept format: `词×词` or `词×词×词` (e.g. `AI×心理健康`, `深度学习×情绪识别`, `LLM×教育`)

Rules:
- Check existing concept folders first: if a close match exists, reuse it
- Check the 已探索概念 table in `~/Desktop/playmore/profile.md` for previously used concepts
- Only create a new folder if no existing concept fits at all
- One paper can only belong to one concept folder (pick the most specific)

List existing concept folders:
```bash
ls "~/Desktop/playmore/"
```

## Step 2 — Create folder if needed

```bash
mkdir "~/Desktop/playmore/{concept}"
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

{FULL editorial comment from playmore-review (2-3 paragraphs)}

### 决定
{Accept / Minor Revision / Major Revision / Reject}

## 摘要

{full abstract}
```

## Step 4 — Save the file

Write the note to:
`~/Desktop/playmore/{concept}/{filename}.md`

## Step 5 — Update INDEX.md

After saving all notes, update `~/Desktop/playmore/INDEX.md`:

```markdown
# Playmore Index

_Last updated: {date}_

{for each concept folder, sorted alphabetically}
## {concept}
{for each note in folder, sorted by date_added desc}
- [{title}](./{concept}/{filename}.md) — {review emoji} {year} · {journal} · {journal_tier}
```

Write the full updated INDEX.md.

## Step 6 — Report back to playmore

Return to `/playmore`:
- `concepts_found`: list of concept labels used
- `top_journals`: journals from ⭐必读 papers
- `top_keywords`: keywords extracted from ⭐必读 paper titles and abstracts
- `must_read_count`: number of ⭐必读 papers

This data is used by `/playmore-profile update`.
