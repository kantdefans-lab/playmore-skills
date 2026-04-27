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
Generate a concise, precise editorial comment in 2-3 paragraphs.

Paragraph 1: Position the paper in the field
- What's the core contribution?
- How does it compare to key papers in background knowledge?
- Is it innovative or incremental?

Paragraph 2: Assess technical quality and limitations
- Are the methods sound?
- Are the experiments convincing?
- What are the main weaknesses?

Paragraph 3 (optional): Recommendation
- Accept / Minor Revision / Major Revision / Reject
- Why?

Style:
- Flowing, natural prose (not bullet points)
- Precise and concrete (not vague)
- Authoritative (grounded in background knowledge)
- No excessive detail (2-3 sentences per paragraph max)

Output format:

## 编辑评论

{paragraph 1}

{paragraph 2}

{paragraph 3 if needed}

### 决定
{Accept / Minor Revision / Major Revision / Reject}
```

## Step 3 — Classify and output

Based on the editorial review, assign classification:

- **⭐ 必读** — Accept or Minor Revision (high contribution, solid quality)
- **📖 可读** — Major Revision (interesting but needs work)
- **💡 了解** — Reject or background context (limited contribution)

## Step 4 — Display summary

After reviewing all papers, group by classification and display:

```
共 {n} 篇 | ⭐ {x} 必读  📖 {y} 可读  💡 {z} 了解

⭐ 必读
1. {title}
   {editorial comment (2-3 paragraphs)}
   
2. {title}
   {editorial comment (2-3 paragraphs)}

📖 可读
1. {title}
   {editorial comment (2-3 paragraphs)}

💡 了解
1. {title}
   {editorial comment (2-3 paragraphs)}
```

## Step 5 — Pass to playmore-notes

Pass the reviewed list (with editorial comments and classifications) to `/playmore-notes`.
