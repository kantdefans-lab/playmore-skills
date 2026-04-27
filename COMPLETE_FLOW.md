# 完整流程 - 从搜索到输出

## 用户输入
```
/playmore AI 心理健康
```

---

## Step 1: playmore-fetch
搜索多个源，返回论文列表

---

## Step 2: playmore-score
对论文打分排序，返回排序后的论文列表

---

## Step 3: playmore-review
**这一步生成AI锐评**

### 3.1 构建背景知识文件（首次）
- 查找领域大牛和高分区论文
- 查找领域大综述
- 生成7步背景知识文件

### 3.2 为每篇论文生成编辑评论
Prompt:
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
Paragraph 2: Assess technical quality and limitations
Paragraph 3 (optional): Recommendation

Style:
- Flowing, natural prose (not bullet points)
- Precise and concrete (not vague)
- Authoritative (grounded in background knowledge)
- No excessive detail (2-3 sentences per paragraph max)
```

### 3.3 输出格式（Step 4）

```
共 n 篇 | ⭐ x 必读  📖 y 可读  💡 z 了解

⭐ 必读
1. LLM-based Therapy Assistant
   作者：Smith, J. | Johnson, M.
   期刊：Nature Mental Health | 分区：SCI Q1 | 发布：2024
   概念：AI×心理健康
   链接：https://...
   
   这篇论文提出了一个新的多模态融合框架用于实时情绪识别，在准确率和推理速度上都超过了现有方法。相比Johnson et al. (2022)的多模态方法（准确率85%），这项工作通过引入生理信号的动态加权机制，将准确率提升到91%，同时保持了实时性（<50ms）。这是该领域的一个重要进展。
   
   方法论上，论文采用了Transformer架构处理时间序列，符合该领域的最佳实践。实验设计严谨，使用了1000+患者的数据集（包含跨文化验证），结果清晰地支持了结论。主要不足是缺少对生理信号数据隐私保护的讨论，这在医疗应用中至关重要。

2. Multimodal Learning for Mental Health
   作者：Johnson, M. | Williams, K.
   期刊：IEEE TMI | 分区：SCI Q2 | 发布：2024
   概念：AI×心理健康
   链接：https://...
   
   {2-3段编辑评论}

📖 可读
1. CNN-based Emotion Recognition
   作者：Zhang, L.
   期刊：ACM TIST | 分区：SCI Q3 | 发布：2023
   概念：深度学习×情绪识别
   链接：https://...
   
   {2-3段编辑评论}

💡 了解
1. Traditional Emotion Detection
   作者：Brown, R.
   期刊：Journal of Psychology | 分区：CSSCI | 发布：2022
   概念：心理学基础
   链接：https://...
   
   {2-3段编辑评论}
```

---

## Step 4: playmore-notes
保存笔记到文件

笔记格式：
```markdown
---
title: LLM-based Therapy Assistant
authors: [Smith, J., Johnson, M.]
year: 2024
journal: Nature Mental Health
journal_tier: SCI Q1
review: ⭐必读
concept: AI×心理健康
keywords: [LLM, therapy, mental health]
sources:
  - platform: OpenAlex
    landing_url: https://...
    pdf_url: https://...
    doi: 10.xxxx/xxxxx
score: 8.5
date_added: 2026-04-27
---

## 编辑评论

这篇论文提出了一个新的多模态融合框架用于实时情绪识别，在准确率和推理速度上都超过了现有方法。相比Johnson et al. (2022)的多模态方法（准确率85%），这项工作通过引入生理信号的动态加权机制，将准确率提升到91%，同时保持了实时性（<50ms）。这是该领域的一个重要进展。

方法论上，论文采用了Transformer架构处理时间序列，符合该领域的最佳实践。实验设计严谨，使用了1000+患者的数据集（包含跨文化验证），结果清晰地支持了结论。主要不足是缺少对生理信号数据隐私保护的讨论，这在医疗应用中至关重要。

### 决定
Accept with Minor Revision

## 摘要

{full abstract}
```

---

## 周报（可选）

```
📊 周报 (2026-04-21 ~ 2026-04-27)

📈 统计数据
  ├─ 新论文：12篇
  ├─ 新概念：3个
  ├─ 已批准概念：2个
  └─ 总概念数：48个

🔥 热门概念
  ├─ Transformer模型 (+2篇)
  │  └─ 新论文：LLM-based Therapy, Multimodal Learning
  ├─ Prompt Engineering (+1篇)
  │  └─ 新论文：Advanced Prompting Techniques
  └─ Multimodal Learning (+1篇)
     └─ 新论文：Cross-modal Fusion

📚 推荐阅读
  ├─ 🔥 LLM-based Therapy Assistant (2024)
  │  └─ 理由：突破性的多模态融合方法
  ├─ 🔥 Multimodal Learning for Mental Health (2024)
  │  └─ 理由：实时性和准确率的完美平衡
  └─ 📖 CNN-based Emotion Recognition (2023)
     └─ 理由：传统方法的最新进展

💡 洞察
  ├─ Transformer模型在心理健康领域应用增加
  ├─ 多模态学习成为新的研究方向
  └─ 生理信号处理开始受关注

📈 趋势分析
  ├─ 热点方向：LLM应用、多模态融合
  ├─ 新兴方向：生理信号处理、个性化治疗
  └─ 冷却方向：单模态CNN方法
```

---

## 关键点

1. **AI锐评在Step 3生成**
   - 使用期刊编辑身份
   - 基于背景知识文件
   - 2-3段流畅文字
   - 精准、简洁、无冗余

2. **Step 4是展示格式**
   - 按分类分组（必读/可读/了解）
   - 每篇论文显示元数据 + 编辑评论
   - 编辑评论是主要内容

3. **笔记保存**
   - 编辑评论 + 摘要
   - 保存到概念文件夹

4. **周报（可选）**
   - 统计数据
   - 热门概念
   - 推荐阅读
   - 洞察和趋势
