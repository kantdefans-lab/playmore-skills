# 动态概念库实现计划

## 用户决定

✅ **概念识别时机**：每次搜索时实时识别（快速反馈）
✅ **自动批准阈值**：置信度 > 0.9 的概念自动批准
✅ **存储位置**：`~/Desktop/playmore/concepts/`
✅ **论文笔记关系**：论文笔记中包含概念链接

---

## 实现架构

### 修改现有流程

```
playmore-fetch
    ↓
playmore-score
    ↓
playmore-review (现有)
    ↓
[新增] playmore-concepts-extract (概念识别)
    ↓
playmore-notes (现有，修改为添加概念链接)
```

### 新增的skill

- `playmore-concepts-extract` - 从论文中识别概念（内部调用）
- `playmore-concepts-review` - 用户审核待批准概念
- `playmore-concepts-organize` - 自动整理层级关系

---

## 数据结构详细设计

### 1. concepts.json 结构

```json
{
  "version": "1.0",
  "last_updated": "2026-04-27T10:30:00Z",
  "seed_concepts": [
    {
      "id": "seed_001",
      "name": "AI×心理健康",
      "description": "AI在心理健康领域的应用",
      "keywords": ["AI", "mental health", "therapy", "心理", "治疗"],
      "created_at": "2026-04-27",
      "paper_count": 0,
      "status": "active"
    }
  ],
  "auto_concepts": [
    {
      "id": "auto_001",
      "name": "Transformer模型",
      "description": "自动识别：在AI×心理健康论文中频繁出现的深度学习架构",
      "keywords": ["transformer", "attention", "BERT", "GPT"],
      "parent_concept_id": "seed_001",
      "confidence": 0.92,
      "discovered_from": ["paper_001", "paper_003"],
      "paper_count": 2,
      "created_at": "2026-04-27",
      "status": "approved",  // pending / approved / rejected
      "auto_approved": true,
      "user_notes": ""
    }
  ],
  "concept_relationships": [
    {
      "source_id": "seed_001",
      "target_id": "auto_001",
      "relation_type": "contains",  // contains / related_to / similar_to
      "strength": 0.95
    }
  ]
}
```

### 2. paper_concepts.json 结构

```json
{
  "paper_001": {
    "title": "LLM-based Therapy Assistant",
    "concepts": [
      {
        "concept_id": "seed_001",
        "concept_name": "AI×心理健康",
        "relevance": 0.95,
        "mention_count": 3,
        "source": "user_defined"
      },
      {
        "concept_id": "auto_001",
        "concept_name": "Transformer模型",
        "relevance": 0.78,
        "mention_count": 2,
        "source": "auto_extracted"
      }
    ],
    "new_concepts_suggested": [
      {
        "name": "Prompt Engineering",
        "relevance": 0.72,
        "confidence": 0.85,
        "context": "论文中详细讨论了如何设计提示词来改进治疗效果",
        "status": "pending"  // pending / approved / rejected
      }
    ],
    "concepts_last_updated": "2026-04-27T10:30:00Z"
  }
}
```

### 3. 概念笔记格式（concepts/auto_001.md）

```markdown
---
concept_id: auto_001
name: Transformer模型
type: auto
status: approved
parent_concept_id: seed_001
related_concepts: []
paper_count: 2
confidence: 0.92
auto_approved: true
created_at: 2026-04-27
last_updated: 2026-04-27
---

# Transformer模型

## 定义
自动识别：在AI×心理健康论文中频繁出现的深度学习架构

## 关键词
transformer, attention, BERT, GPT

## 核心论文
- [[2024_smith_llm-therapy]] (relevance: 0.95)
- [[2023_zhang_emotion-detection]] (relevance: 0.78)

## 相关概念
- [[seed_001|AI×心理健康]]

## 发现历史
- 2026-04-27: 从论文 paper_001 中自动识别（confidence: 0.92）
- 2026-04-27: 从论文 paper_003 中再次识别（confidence: 0.88）
- 2026-04-27: 自动批准（confidence > 0.9）
```

---

## 工作流程详细设计

### Step 1: 用户初始化

**触发**：用户首次使用或修改config.json

**config.json 格式**：

```json
{
  "keywords": ["AI", "mental health", "therapy"],
  "negative_keywords": ["medical imaging"],
  "seed_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy", "心理", "治疗"],
      "description": "AI在心理健康领域的应用"
    },
    {
      "name": "深度学习×情绪识别",
      "keywords": ["deep learning", "emotion", "recognition", "情绪", "识别"],
      "description": "用深度学习识别人类情绪"
    }
  ]
}
```

**系统动作**：
1. 读取config.json中的seed_concepts
2. 为每个种子概念生成唯一ID（seed_001, seed_002, ...）
3. 创建concepts.json
4. 为每个种子概念创建笔记文件（concepts/seed_001.md）

---

### Step 2: 搜索和打分（现有流程）

```
/playmore AI 心理健康
    ↓
playmore-fetch (搜索)
    ↓
playmore-score (打分)
    ↓
playmore-review (AI点评)
```

**输出**：scored_papers.json

```json
[
  {
    "title": "LLM-based Therapy Assistant",
    "abstract": "...",
    "authors": [...],
    "score": 8,
    "review": "🔥必读",
    "sources": [...]
  }
]
```

---

### Step 3: 概念识别（新增）

**触发**：playmore-review完成后，自动调用playmore-concepts-extract

**输入**：scored_papers.json

**Claude Prompt**：

```
你是一个学术概念分类专家。

已有的概念库：
{
  "seed_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy", "心理", "治疗"]
    },
    {
      "name": "深度学习×情绪识别",
      "keywords": ["deep learning", "emotion", "recognition", "情绪", "识别"]
    }
  ]
}

论文列表：
[
  {
    "title": "LLM-based Therapy Assistant",
    "abstract": "...",
    "keywords": ["LLM", "therapy", "mental health"]
  }
]

任务：
1. 对每篇论文，判断与现有概念的相关度（0-1）
2. 识别论文中的新概念（不在现有库中的）
3. 为新概念计算置信度（0-1）

输出 JSON 数组：
[
  {
    "paper_title": "LLM-based Therapy Assistant",
    "matched_concepts": [
      {
        "concept_name": "AI×心理健康",
        "relevance": 0.95,
        "reason": "论文主要讨论LLM在心理治疗中的应用"
      }
    ],
    "new_concepts": [
      {
        "name": "Prompt Engineering",
        "description": "设计有效的提示词来指导AI模型的行为",
        "keywords": ["prompt", "instruction", "engineering", "提示词"],
        "parent_concept": "AI×心理健康",
        "relevance": 0.78,
        "confidence": 0.85,
        "context": "论文中详细讨论了如何设计提示词来改进治疗效果"
      },
      {
        "name": "Multimodal Learning",
        "description": "融合多种模态（文本、语音、图像）的学习方法",
        "keywords": ["multimodal", "fusion", "cross-modal"],
        "parent_concept": "AI×心理健康",
        "relevance": 0.65,
        "confidence": 0.72,
        "context": "论文提到使用语音和文本的多模态输入"
      }
    ]
  }
]
```

**系统动作**：
1. 调用Claude进行概念识别
2. 更新paper_concepts.json
3. 对于confidence > 0.9的概念，自动添加到concepts.json（status: approved）
4. 对于confidence ≤ 0.9的概念，标记为pending
5. 为新批准的概念创建笔记文件

---

### Step 4: 论文笔记生成（修改现有流程）

**修改**：playmore-notes在生成论文笔记时，添加概念链接

**论文笔记格式**（AI×心理健康/2024_smith_llm-therapy.md）：

```markdown
---
title: LLM-based Therapy Assistant
authors: [Smith, J., ...]
year: 2024
journal: Nature Mental Health
review: 🔥必读
concepts:
  - AI×心理健康 (relevance: 0.95)
  - Transformer模型 (relevance: 0.78)
  - Prompt Engineering (relevance: 0.78)
---

# LLM-based Therapy Assistant

## 概念链接
- [[AI×心理健康]] (核心概念)
- [[Transformer模型]] (技术基础)
- [[Prompt Engineering]] (关键方法)

## 核心贡献
...

## 研究方法
...

## 实验结果
...
```

**系统动作**：
1. 从paper_concepts.json读取该论文的概念
2. 在论文笔记的frontmatter中添加concepts字段
3. 在笔记开头添加"概念链接"section
4. 使用wikilink格式：`[[概念名]]`

---

### Step 5: 用户审核待批准概念

**触发**：用户手动调用或定期检查

**命令**：

```bash
/playmore-concepts-review
```

**输出**：

```
待审核概念（confidence ≤ 0.9）：

1. Multimodal Learning (0.72 confidence)
   - 发现于：paper_001, paper_003
   - 描述：融合多种模态的学习方法
   - 上级概念：AI×心理健康
   - 相关论文：2篇
   
   [A] 批准  [R] 拒绝  [E] 编辑  [S] 跳过

2. Reinforcement Learning in Therapy (0.65 confidence)
   - 发现于：paper_002
   - 描述：使用强化学习优化治疗策略
   - 上级概念：AI×心理健康
   - 相关论文：1篇
   
   [A] 批准  [R] 拒绝  [E] 编辑  [S] 跳过
```

**用户操作**：
- `A` - 批准，添加到concepts.json
- `R` - 拒绝，标记为rejected
- `E` - 编辑，修改描述/关键词后批准
- `S` - 跳过，保持pending状态

**系统动作**：
1. 更新concepts.json中的status
2. 如果批准，创建概念笔记文件
3. 更新paper_concepts.json中的status

---

### Step 6: 自动整理层级关系（可选）

**触发**：用户手动调用或定期触发

**命令**：

```bash
/playmore-organize-concepts
```

**Claude Prompt**：

```
你是一个知识图谱设计专家。

现有概念列表：
[
  {
    "name": "AI×心理健康",
    "type": "seed",
    "paper_count": 12
  },
  {
    "name": "Transformer模型",
    "type": "auto",
    "paper_count": 9
  },
  {
    "name": "Prompt Engineering",
    "type": "auto",
    "paper_count": 7
  },
  {
    "name": "Multimodal Learning",
    "type": "auto",
    "paper_count": 4
  }
]

任务：
1. 识别概念的层级关系（parent/child）
2. 识别概念的相似性（是否应该合并）
3. 建议概念的重组

输出 JSON：
{
  "hierarchy_suggestions": [
    {
      "parent": "AI×心理健康",
      "children": ["Transformer模型", "Prompt Engineering", "Multimodal Learning"]
    }
  ],
  "merge_suggestions": [
    {
      "concepts": ["Concept A", "Concept B"],
      "reason": "这两个概念高度重叠",
      "recommended_name": "Merged Concept"
    }
  ],
  "new_parent_suggestions": [
    {
      "concept": "Transformer模型",
      "suggested_parent": "Deep Learning Architectures",
      "reason": "可以创建一个新的上级概念来组织所有深度学习架构"
    }
  ]
}
```

**系统动作**：
1. 调用Claude进行层级分析
2. 显示建议给用户
3. 用户确认后更新concepts.json中的parent_concept_id

---

## 概念笔记索引生成

**触发**：每次概念库更新后

**生成文件**：concepts/INDEX.md

```markdown
# 概念库索引

## 种子概念（用户定义）
- [[seed_001|AI×心理健康]] - 12篇论文
- [[seed_002|深度学习×情绪识别]] - 8篇论文

## 自动识别概念

### 已批准（Approved）
- [[auto_001|Transformer模型]] - 9篇论文 (confidence: 0.92)
- [[auto_002|Prompt Engineering]] - 7篇论文 (confidence: 0.85)
- [[auto_003|Multimodal Learning]] - 4篇论文 (confidence: 0.72)

### 待审核（Pending）
- Reinforcement Learning in Therapy - 1篇论文 (confidence: 0.65)

### 已拒绝（Rejected）
- Graph Neural Networks - 拒绝原因：与现有概念重复

## 最近更新
- 2026-04-27: 新增 Prompt Engineering (auto_002)
- 2026-04-27: 批准 Multimodal Learning (auto_003)
- 2026-04-27: 创建种子概念
```

---

## 关键实现细节

### 1. 概念ID生成规则

```python
# 种子概念：seed_001, seed_002, ...
# 自动概念：auto_001, auto_002, ...

def generate_concept_id(concept_type, index):
    return f"{concept_type}_{index:03d}"
```

### 2. 置信度计算

```python
# 基于以下因素：
# - 关键词匹配度（0-0.4）
# - 在多篇论文中出现（每篇+0.1，最多0.3）
# - Claude的评估（0-0.3）

confidence = keyword_match * 0.4 + paper_frequency * 0.3 + claude_score * 0.3
```

### 3. 自动批准逻辑

```python
if confidence > 0.9:
    status = "approved"
    auto_approved = True
else:
    status = "pending"
    auto_approved = False
```

### 4. 论文笔记中的概念链接

```markdown
# 格式1：简单链接
[[概念名]]

# 格式2：带标签的链接
[[概念名|显示文本]]

# 格式3：带相关度的链接
[[概念名]] (relevance: 0.95)

# 在frontmatter中
concepts:
  - AI×心理健康 (relevance: 0.95)
  - Transformer模型 (relevance: 0.78)
```

---

## 存储结构最终版

```
~/Desktop/playmore/
├── config.json                       # 用户配置 + 种子概念
├── concepts.json                     # 概念库元数据
├── paper_concepts.json               # 论文-概念映射
├── schedule.json                     # 定时任务
├── INDEX.md                          # 论文索引
├── concepts/
│   ├── INDEX.md                      # 概念库总索引
│   ├── seed_001.md                   # AI×心理健康
│   ├── seed_002.md                   # 深度学习×情绪识别
│   ├── auto_001.md                   # Transformer模型
│   ├── auto_002.md                   # Prompt Engineering
│   └── ...
├── AI×心理健康/
│   ├── 2024_smith_llm-therapy.md
│   └── ...
├── 深度学习×情绪识别/
│   ├── 2023_zhang_emotion-detection.md
│   └── ...
└── ...
```

---

## 实现优先级

### Phase 1（必需）- 基础概念库
- [ ] 修改config.json格式，支持seed_concepts
- [ ] 创建concepts.json和paper_concepts.json
- [ ] 实现playmore-concepts-extract skill
- [ ] 实现自动批准逻辑（confidence > 0.9）
- [ ] 修改playmore-notes，添加概念链接
- [ ] 生成概念笔记文件

### Phase 2（重要）- 用户交互
- [ ] 实现playmore-concepts-review skill
- [ ] 实现概念批准/拒绝逻辑
- [ ] 生成concepts/INDEX.md

### Phase 3（可选）- 高级功能
- [ ] 实现playmore-organize-concepts skill
- [ ] 实现概念去重检测
- [ ] 实现概念编辑功能

---

## 与跨天去重的集成

**跨天去重**应该在schedule.json中实现：

```json
{
  "schedule": [...],
  "history": {
    "2026-04-27": [
      {"arxiv_id": "2404.12345", "title": "...", "doi": "..."},
      {"arxiv_id": "2404.12346", "title": "...", "doi": "..."}
    ],
    "2026-04-26": [...]
  }
}
```

**去重逻辑**：
1. 搜索时，检查history中是否存在相同的论文（按DOI或标题相似度）
2. 如果存在，标记为"已推荐"，可选择是否再次推荐
3. 新论文添加到history中

