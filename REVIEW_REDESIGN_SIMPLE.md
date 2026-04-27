# AI锐评重设计 - 简洁版本

## 核心原则

编辑评论应该是**2-3段流畅的文字**，而不是分层的结构。

- 第1段：定位论文在领域中的位置，指出核心创新
- 第2段：评估技术质量和主要不足
- 第3段（可选）：给出明确的决定和建议

## 例子1：高质量论文（Accept）

```
## 编辑评论

这篇论文提出了一个新的多模态融合框架用于实时情绪识别，在准确率和推理速度上都超过了现有方法。相比Johnson et al. (2022)的多模态方法（准确率85%），这项工作通过引入生理信号的动态加权机制，将准确率提升到91%，同时保持了实时性（<50ms）。这是该领域的一个重要进展。

方法论上，论文采用了Transformer架构处理时间序列，符合该领域的最佳实践。实验设计严谨，使用了1000+患者的数据集（包含跨文化验证），结果清晰地支持了结论。主要不足是缺少对生理信号数据隐私保护的讨论，这在医疗应用中至关重要。

### 决定
Accept with Minor Revision
```

## 例子2：中等质量论文（Major Revision）

```
## 编辑评论

这篇论文尝试使用LLM进行心理健康评估，这是一个重要的研究方向。但相比Smith et al. (2023)的开创性工作，这项工作的创新有限，主要是在中文数据集上的重复验证。虽然中文心理健康AI应用有市场需求，但论文的准确率78%低于该领域的标准（>80%），样本量200个患者也不足以支撑医疗应用。

方法论上，论文直接采用了现有的LLM（GPT-3.5），没有进行任何微调或优化。建议作者扩大数据集规模到至少1000个患者，进行LLM微调以提升准确率，并加入多模态数据（语音、生理信号）来增强创新性。

### 决定
Major Revision Required
```

## 例子3：低质量论文（Reject）

```
## 编辑评论

这篇论文尝试使用CNN进行情绪识别，但这个方向已经被广泛研究，且论文的方法论和结果都不足以对该领域做出新的贡献。相比Johnson et al. (2022)的多模态方法（准确率85%）和Smith et al. (2023)的LLM方法（准确率90%+），这项工作的单模态CNN方法（准确率72%）已经落后。论文的参考文献主要来自2018-2020年的工作，缺少对最近5年研究进展的了解。

实验设计简陋，只使用了100个患者的数据集，这对于医疗应用来说完全不足。方法论上采用了标准的CNN架构，没有任何优化或创新。这项工作的准确率和方法论都不足以支撑任何实际应用。

### 决定
Reject
```

## Prompt模板

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
Generate a concise editorial comment in 2-3 paragraphs.

Paragraph 1: Position the paper
- What's the core contribution?
- How does it compare to key papers in background knowledge?
- Is it innovative or incremental?

Paragraph 2: Assess quality and limitations
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
- 2-3 sentences per paragraph max

Output:

## 编辑评论

{paragraph 1}

{paragraph 2}

{paragraph 3 if needed}

### 决定
{Accept / Minor Revision / Major Revision / Reject}
```

## 笔记模板

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
score: {total score}
date_added: {today YYYY-MM-DD}
---

## 编辑评论

{full editorial comment (2-3 paragraphs)}

### 决定
{Accept / Minor Revision / Major Revision / Reject}

## 摘要

{full abstract}
```
