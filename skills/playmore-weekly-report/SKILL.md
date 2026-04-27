---
name: playmore-weekly-report
description: Generate weekly summary of new papers, concepts, trends, and insights. Show statistics and recommendations.
argument-hint: "[--week N|--month|--custom DATE_RANGE]"
---

# Playmore Weekly Report — Trends & Insights

Generate weekly summaries of research activity, trends, and insights.

## Arguments

$ARGUMENTS can be:
- (empty) — current week
- `--week N` — N weeks ago
- `--month` — current month
- `--custom {start_date} {end_date}` — custom date range

## Step 1 — Collect data

For the specified time period, gather:
- New papers added
- New concepts discovered
- Approved concepts
- Concept trends (which concepts got more papers)
- Top papers by citations
- Emerging research directions

## Step 2 — Generate report

Output format:

```
📊 周报 ({date_range})

📈 统计数据
  ├─ 新论文：{count}篇
  ├─ 新概念：{count}个
  ├─ 已批准概念：{count}个
  └─ 总概念数：{count}个

🔥 热门概念
  ├─ {concept_1} (+{count}篇)
  │  └─ 新论文：{paper_titles}
  ├─ {concept_2} (+{count}篇)
  │  └─ 新论文：{paper_titles}
  └─ {concept_3} (+{count}篇)
     └─ 新论文：{paper_titles}

📚 推荐阅读
  ├─ 🔥 {paper_title_1} ({year})
  │  └─ 理由：{reason}
  ├─ 🔥 {paper_title_2} ({year})
  │  └─ 理由：{reason}
  └─ 📖 {paper_title_3} ({year})
     └─ 理由：{reason}

📉 冷门概念
  ├─ {concept_1} (0篇新论文)
  │  └─ 建议：考虑归档
  ├─ {concept_2} (0篇新论文)
  │  └─ 建议：考虑归档
  └─ ...

💡 洞察
  ├─ {insight_1}
  ├─ {insight_2}
  └─ {insight_3}

📈 趋势分析
  ├─ 热点方向：{direction_1}, {direction_2}
  ├─ 新兴方向：{direction_3}, {direction_4}
  └─ 冷却方向：{direction_5}
```

## Step 3 — Analyze trends

For each concept, calculate:
- Paper count change (week-over-week)
- Growth rate
- Trend direction (↑ increasing, → stable, ↓ decreasing)

Identify:
- Emerging concepts (new, high growth)
- Hot concepts (high paper count, positive growth)
- Stable concepts (consistent papers)
- Cold concepts (no new papers)

## Step 4 — Generate insights

Use Claude to generate 3-5 insights:

```
你是一个学术研究分析师。

基于以下数据，生成3-5个关键洞察：

新论文数：{count}
新概念数：{count}
热门概念：{concepts}
论文主题：{topics}

洞察应该：
1. 识别新的研究方向
2. 指出领域的发展趋势
3. 发现潜在的研究机会
4. 提出后续研究建议

输出格式：
- 洞察1：{insight}
- 洞察2：{insight}
- ...
```

## Step 5 — Save report

Save to:
`C:\Users\kantd\Desktop\playmore\reports\weekly_{date}.md`

Format:
```markdown
---
title: Weekly Report - {date_range}
date: {today}
period: {week_number}
---

## 统计数据

{statistics}

## 热门概念

{trending concepts}

## 推荐阅读

{recommended papers}

## 冷门概念

{cold concepts}

## 洞察

{insights}

## 趋势分析

{trend analysis}

## 后续建议

{recommendations}
```

## Step 6 — Display report

Output formatted report to console.

Optionally save as:
- Markdown file
- HTML file
- PDF file

## Token consumption

- Data collection: 0 tokens
- Trend analysis: 0 tokens
- Insight generation: 500-800 tokens (optional Claude)
- Total: 500-800 tokens per report
