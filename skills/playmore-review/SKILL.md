---
name: playmore-review
description: Journal editor-style AI review of scored papers. Uses background knowledge files for domain context. Generates precise, flowing editorial comments. Called by playmore after playmore-score.
argument-hint: ""
---

# Playmore Review — Journal Editor Mode

Takes the ranked paper list from playmore-score and generates journal editor-style reviews.

## Step 1 — Identify concept and load/build background knowledge

For each paper, extract its concept from title + abstract + keywords.

Check if background knowledge file exists:
```
C:\Users\kantd\Desktop\playmore\background_knowledge\{concept}_background.md
```

**If file exists:** Load it as reference material.

**If file not exist:** Build it using:
1. Query OpenAlex/Semantic Scholar for domain experts in this concept
2. Fetch their top-tier papers (2024-2020, Q1/Q2 journals)
3. Fetch domain surveys/reviews
4. Extract key sections from PDFs using Marker (local, 0 tokens)
5. Generate 7-step background knowledge file:
   - Domain positioning
   - Core literature (key papers + authors)
   - Theoretical frameworks
   - Research frontiers
   - Key controversies
   - Evaluation standards
   - Structured summary

Save to `C:\Users\kantd\Desktop\playmore\background_knowledge\{concept}_background.md`

## Step 2 — Generate journal editor review

For each paper, use this prompt structure:

```
You are a journal editor with 15+ years of experience in {concept}.

【Background Knowledge Reference】
{background_knowledge_summary}

【Paper to Review】
Title: {title}
Authors: {authors}
Year: {year}
Journal: {journal}
Citations: {cited_by}

Abstract:
{abstract}

{method_summary if available}

{key_results if available}

【Review Instructions】
Generate a professional, precise editorial comment. 

The review should:
1. Quickly position the paper in the field (vs. key papers in background knowledge)
2. Precisely identify innovations and limitations (based on evaluation standards)
3. Give clear recommendation (Accept / Minor Revision / Major Revision / Reject)

Style:
- Flowing, not fragmented (avoid excessive point-breaking)
- Precise, not verbose (each point has concrete support)
- Authoritative, not arbitrary (grounded in background knowledge and data)

Output format:

## 编辑评论

### 总体评价
[1-2 sentences]

### 创新性和意义
[2-3 sentences]

### 技术质量
[2-3 sentences]

### 与相关工作的关系
[2-3 sentences]

### 应用前景和影响
[1-2 sentences]

### 主要问题
[2-3 key issues, 1-2 sentences each]

### 建议
[1-2 sentences]

### 决定
[Accept / Minor Revision / Major Revision / Reject]
```

## Step 3 — Classify and output

Based on the editorial review, assign classification:

- **⭐ 必读** — Accept or Minor Revision (high contribution, solid quality)
- **📖 可读** — Major Revision (interesting but needs work)
- **💡 了解** — Reject or background context (limited contribution)

Output format:
```
{emoji} {title}
编辑评论：{editorial comment}
```

## Step 4 — Display summary

After reviewing all papers, print:

```
共 {n} 篇 | ⭐ {x} 必读  📖 {y} 可读  💡 {z} 了解

⭐ 必读
1. {title} — {decision}
   {1-sentence summary}

📖 可读
...

💡 了解
...
```

## Step 5 — Pass to playmore-notes

Pass the reviewed list (with editorial comments) to `/playmore-notes`.
