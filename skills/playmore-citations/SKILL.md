---
name: playmore-citations
description: Show paper citation relationships and networks. Display citing papers, cited papers, and citation statistics.
argument-hint: "[paper_id|concept] [--network|--top-cited|--depth N]"
---

# Playmore Citations — Citation Network

Display citation relationships and build citation networks.

## Arguments

$ARGUMENTS can be:
- `{paper_id}` — show citations for a specific paper
- `{concept}` — show top-cited papers in a concept
- `--network` — show full citation network (depth 2)
- `--top-cited` — show most-cited papers in concept
- `--depth N` — network depth (1-3)

## Step 1 — Fetch citation data

For a given paper, fetch:
1. Papers it cites (references)
2. Papers that cite it (citing papers)

Data sources (0 tokens):
- Semantic Scholar API
- OpenAlex API
- CrossRef API
- Local paper library

## Step 2 — Display citation statistics

Output:
```
📊 {paper_title} 的引用统计

被引用次数：{count}次
  ├─ 2024年：{count}次
  ├─ 2023年：{count}次
  └─ 2022年：{count}次

引用的论文数：{count}篇
  ├─ 2023年：{count}篇
  ├─ 2022年：{count}篇
  └─ 2021年：{count}篇

影响力排名：领域前{percentile}%
```

## Step 3 — Show citation tree

Display direct citations:

```
📖 这篇论文引用了：
  ├─ 论文A (2023) - 被引用100次
  │  └─ 关键性：高
  ├─ 论文B (2022) - 被引用80次
  │  └─ 关键性：高
  └─ 论文C (2021) - 被引用30次
     └─ 关键性：中

📚 引用这篇论文的：
  ├─ 论文D (2024) - 新论文
  ├─ 论文E (2024) - 新论文
  └─ 论文F (2024) - 新论文
```

## Step 4 — Build citation network

If `--network` flag, build full network:

```
        论文C(2022)
           ↑
        论文B(2023)
           ↑
        论文A(2024) ──→ 论文G(2024)
           ↑
        论文E(2023)
           ↑
        论文F(2021)
```

Network depth:
- `--depth 1` — only direct citations
- `--depth 2` — citations of citations
- `--depth 3+` — deeper network

## Step 5 — Find top-cited papers

Command: `/playmore-citations --top-cited --concept {concept}`

Output:
```
🔥 {concept} 领域最有影响力的论文：

1. 论文A (2020)
   ├─ 被引用：500次
   ├─ 影响力：★★★★★
   └─ 推荐指数：必读

2. 论文B (2021)
   ├─ 被引用：400次
   ├─ 影响力：★★★★★
   └─ 推荐指数：必读

3. 论文C (2022)
   ├─ 被引用：300次
   ├─ 影响力：★★★★
   └─ 推荐指数：推荐
```

## Step 6 — Save citation analysis

Save to:
`C:\Users\kantd\Desktop\playmore\citations\{paper_id}_citations.md`

Format:
```markdown
---
title: {paper_title} - Citation Analysis
paper_id: {id}
date: {today}
---

## 引用统计

{citation statistics}

## 引用关系

{citation tree}

## 网络分析

{network analysis if available}

## 关键论文

{top cited papers in network}
```

## Token consumption

- Citation fetching: 0 tokens (API calls only)
- Network building: 0 tokens (local processing)
- Analysis: 0-500 tokens (optional Claude analysis)
