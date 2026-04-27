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

For each paper, generate TWO versions of editorial comment:

### Version A: Full editorial comment (for notes)
Use this prompt structure:

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

### Version B: Simplified editorial comment (for daily report)
Extract 2-3 key sentences from Version A that capture the essence.

Example:
- Full: "这篇论文提出了一个新的多模态融合框架用于实时情绪识别，在准确率和推理速度上都超过了现有方法。相比Johnson et al. (2022)的多模态方法（准确率85%），这项工作通过引入生理信号的动态加权机制，将准确率提升到91%，同时保持了实时性（<50ms）。这是该领域的一个重要进展。方法论上，论文采用了Transformer架构处理时间序列，符合该领域的最佳实践。实验设计严谨，使用了1000+患者的数据集（包含跨文化验证），结果清晰地支持了结论。主要不足是缺少对生理信号数据隐私保护的讨论，这在医疗应用中至关重要。"

- Simplified: "提出新的多模态融合框架，准确率从85%提升到91%。方法论严谨，但缺少隐私保护讨论。"

## Step 3 — Generate overall synthesis

After reviewing all papers, generate a focused academic synthesis in 3 paragraphs grounded in background knowledge:

```
你是一个学术研究分析师。

基于背景知识文件和这些论文的编辑评论，生成一个学术性的整体分析。

背景知识：
{background_knowledge_summary}

论文列表：
{all_papers_with_comments}

请用3段进行学术讨论（不涉及方向和展望）：

第1段：与核心文献的关系
- 这些论文如何验证、推进或挑战背景知识中的关键论文（Smith et al. 2023, Johnson et al. 2022等）？
- 具体的性能对比是什么？
- 是否存在方法论上的本质突破？

第2段：对理论框架的贡献
- 这些论文如何验证或推进背景知识中的理论框架？
- 是否提出了新的理论假设？
- 在评估标准（准确率、鲁棒性、实时性、可解释性等）上的表现如何？

第3段：在关键争议中的立场
- 这些论文在背景知识中的关键争议（如"AI能否替代心理医生"、"隐私与性能的权衡"）中采取了什么立场？
- 是否提供了新的证据或视角？
- 存在哪些未解决的根本问题？

要求：
- 具体引用背景知识中的论文和框架
- 使用具体的数据和指标
- 学术性的论证，避免泛泛而谈
- 每段3-4句话
```

Output: **整体锐评思考** (3 paragraphs, academic focus)

## Step 4 — Classify and output

Based on the editorial review, assign classification:

- **⭐ 必读** — Accept or Minor Revision (high contribution, solid quality)
- **📖 可读** — Major Revision (interesting but needs work)
- **💡 了解** — Reject or background context (limited contribution)

## Step 5 — Display summary (States of Mind Map)

After reviewing all papers, display in this format:

```
## 关键词
{keywords from all papers}

## 简要摘要
本次搜索"{search_keywords}"共找到{n}篇论文，主要集中在{main_directions}...

## 整体锐评思考
{global_synthesis_comment}

## States of Mind 地图

### ⭐ 必读 ({count}篇)
1. {title} ({authors}, {year})
   - 核心贡献：{core_contribution}
   - 编辑评论：{simplified_comment_2-3_sentences}
   
2. {title} ({authors}, {year})
   - 核心贡献：{core_contribution}
   - 编辑评论：{simplified_comment_2-3_sentences}

### 📖 可读 ({count}篇)
1. {title} ({authors}, {year})
   - 核心贡献：{core_contribution}
   - 编辑评论：{simplified_comment_2-3_sentences}

### 💡 了解 ({count}篇)
...
```

## Step 6 — Pass to playmore-notes

Pass the reviewed list with:
- Full editorial comments (for notes)
- Simplified editorial comments (for display)
- Classifications
- Global synthesis

to `/playmore-notes`.
