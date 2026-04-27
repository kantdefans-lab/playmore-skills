---
name: playmore-concepts
description: Manage concept library. Extract concepts from papers, review pending concepts, browse and search concepts, visualize concept relationships.
argument-hint: "[list|search|review|visualize] or concept name"
---

# Playmore Concepts — Concept Library Management

Manages the dynamic concept library that grows as papers are analyzed.

## Arguments

$ARGUMENTS can be:
- `list` — show all concepts
- `search {keyword}` — search concepts
- `review` — review pending concepts
- `visualize` — show concept relationship graph
- `{concept_name}` — show details of a specific concept

## Step 1 — List all concepts

Command: `/playmore-concepts list`

Output:
```
📚 概念库（{n}个概念）

🌱 种子概念（{m}个）
  ├─ {seed_concept_1} ({paper_count}篇论文)
  └─ {seed_concept_2} ({paper_count}篇论文)

🤖 自动概念（{auto_count}个）
  ├─ 🔥 热门概念（命中率>5）
  │  ├─ {concept_1} ({paper_count}篇)
  │  └─ ...
  │
  ├─ 📖 常见概念（命中率2-5）
  │  ├─ {concept_2} ({paper_count}篇)
  │  └─ ...
  │
  └─ 💡 新概念（命中率1）
     ├─ {concept_3} ({paper_count}篇)
     └─ ...
```

## Step 2 — Search concepts

Command: `/playmore-concepts search {keyword}`

Output:
```
搜索结果："{keyword}"

✓ {concept_name}
  ├─ 类型：{自动概念 / 种子概念}
  ├─ 置信度：{confidence}
  ├─ 论文数：{count}
  ├─ 相关概念：{related_concepts}
  └─ 最后更新：{date}
```

## Step 3 — Review pending concepts

Command: `/playmore-concepts review`

Display pending concepts (confidence 0.6-0.9) that need user approval:

```
待审核概念（{n}个）

1️⃣  {concept_name}
   ├─ 置信度：{confidence}
   ├─ 发现于：{paper_count}篇论文
   ├─ 描述：{auto_generated_description}
   ├─ 关键词：{keywords}
   ├─ 上级概念：{parent_concept}
   │
   └─ 操作：
      [A] 批准  [R] 拒绝  [E] 编辑  [S] 跳过
```

User can:
- **A** — Approve concept (add to library)
- **R** — Reject concept (discard)
- **E** — Edit concept (modify name, description, keywords)
- **S** — Skip (review later)

After user action, update concept status and save to:
`C:\Users\kantd\Desktop\playmore\concepts\{concept_id}.json`

## Step 4 — Show concept details

Command: `/playmore-concepts {concept_name}`

Output:
```
📌 {concept_name}

基本信息
  ├─ 类型：{种子概念 / 自动概念}
  ├─ 置信度：{confidence}
  ├─ 论文数：{count}
  ├─ 创建日期：{date}
  └─ 最后更新：{date}

描述
{description}

关键词
{keywords}

相关概念
  ├─ 上级概念：{parent_concept}
  └─ 相关概念：{related_concepts}

论文列表
  1. {paper_title} ({year})
  2. ...
```

## Step 5 — Visualize concept relationships

Command: `/playmore-concepts visualize`

Generate ASCII concept relationship graph:

```
                    {root_concept}
                         |
                    _____|_____
                   |           |
            {concept_1}    {concept_2}
                |              |
                |______________|
                       |
                {concept_3}
                       |
            {concept_4}  {concept_5}
```

Or generate Mermaid diagram for better visualization.

## Storage

Concepts stored in:
- `C:\Users\kantd\Desktop\playmore\concepts\` — individual concept files
- `C:\Users\kantd\Desktop\playmore\concepts\INDEX.json` — concept index and relationships

Concept file format:
```json
{
  "id": "concept_001",
  "name": "AI×心理健康",
  "type": "seed",
  "confidence": 1.0,
  "description": "...",
  "keywords": ["AI", "心理健康", "..."],
  "parent_concept": null,
  "related_concepts": ["concept_002", "concept_003"],
  "paper_count": 12,
  "created_at": "2026-04-27",
  "updated_at": "2026-04-27",
  "status": "approved"
}
```
