# 动态概念库系统设计

## 核心理念

**用户定义种子概念 → AI自动识别和扩充 → 自动整理和维护**

不同于 dailypaper-skills 的固定16分类，这个系统是：
- **用户驱动**：用户定义初始概念
- **AI驱动**：AI从论文中自动识别相关概念
- **动态演进**：概念库随着论文积累而自动扩充
- **自我整理**：AI自动识别概念关系和层级

---

## 数据结构

### 1. 概念库配置（concepts.json）

```json
{
  "seed_concepts": [
    {
      "id": "concept_001",
      "name": "AI×心理健康",
      "description": "AI在心理健康领域的应用",
      "keywords": ["AI", "mental health", "therapy", "心理", "治疗"],
      "created_at": "2026-04-27",
      "user_defined": true
    },
    {
      "id": "concept_002", 
      "name": "深度学习×情绪识别",
      "description": "用深度学习识别人类情绪",
      "keywords": ["deep learning", "emotion", "recognition", "情绪", "识别"],
      "created_at": "2026-04-27",
      "user_defined": true
    }
  ],
  "auto_concepts": [
    {
      "id": "concept_auto_001",
      "name": "Transformer模型",
      "description": "自动识别：在AI×心理健康论文中频繁出现",
      "keywords": ["transformer", "attention", "BERT"],
      "parent_concepts": ["concept_001"],
      "confidence": 0.85,
      "discovered_from": ["paper_001", "paper_003", "paper_005"],
      "created_at": "2026-04-27",
      "user_defined": false,
      "status": "pending_review"  // pending_review / approved / rejected
    }
  ],
  "concept_relationships": [
    {
      "source": "concept_001",
      "target": "concept_auto_001",
      "relation_type": "uses",  // uses / related_to / parent_of / child_of
      "strength": 0.9
    }
  ]
}
```

### 2. 论文-概念映射（paper_concepts.json）

```json
{
  "paper_001": {
    "title": "LLM-based Therapy Assistant",
    "concepts": [
      {
        "concept_id": "concept_001",
        "mention_count": 3,
        "relevance": 0.95,
        "source": "user_defined"  // user_defined / auto_extracted
      },
      {
        "concept_id": "concept_auto_001",
        "mention_count": 2,
        "relevance": 0.78,
        "source": "auto_extracted"
      }
    ],
    "new_concepts_suggested": [
      {
        "name": "Prompt Engineering",
        "relevance": 0.72,
        "context": "论文中讨论了如何设计提示词来改进治疗效果"
      }
    ]
  }
}
```

### 3. 概念笔记（concepts/{concept_id}.md）

```markdown
---
concept_id: concept_001
name: AI×心理健康
type: seed  # seed / auto
status: active  # active / archived
parent_concepts: []
child_concepts: [concept_auto_001, concept_auto_002]
related_concepts: [concept_002]
paper_count: 12
created_at: 2026-04-27
last_updated: 2026-04-28
---

# AI×心理健康

## 定义
用户定义：AI在心理健康领域的应用

## 核心论文
- paper_001: LLM-based Therapy Assistant (⭐必读)
- paper_003: Emotion Recognition with Deep Learning (📖可读)

## 自动识别的子概念
- Transformer模型 (0.85 confidence)
- Prompt Engineering (0.72 confidence)
- Multimodal Learning (0.68 confidence)

## 相关概念
- 深度学习×情绪识别

## 演进历史
- 2026-04-27: 创建种子概念
- 2026-04-28: 自动识别3个子概念
```

---

## 工作流程

### Phase 1: 用户定义种子概念

**触发**：用户在 config.json 中定义

```json
{
  "seed_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy"]
    }
  ]
}
```

**系统动作**：
1. 生成 concept_id
2. 创建概念笔记文件
3. 初始化 concepts.json

---

### Phase 2: 论文搜索和打分（现有流程）

**现有**：playmore-fetch → playmore-score → playmore-review

**新增**：在 playmore-review 中调用概念提取

---

### Phase 3: 自动概念识别（新增）

**触发**：playmore-review 处理每篇论文时

**步骤**：

```
1. 读取论文标题、摘要、关键词
2. 读取 concepts.json 的所有概念
3. 调用 Claude 进行概念匹配和扩充
   - 匹配现有概念（relevance score）
   - 识别新概念（name + description + keywords）
   - 识别概念关系（parent/child/related）
4. 保存到 paper_concepts.json
5. 更新 concepts.json（新概念标记为 pending_review）
```

**Claude Prompt 示例**：

```
你是一个学术概念分类专家。

已有的概念库：
- AI×心理健康（keywords: AI, mental health, therapy）
- 深度学习×情绪识别（keywords: deep learning, emotion, recognition）

论文信息：
- 标题：{title}
- 摘要：{abstract}
- 关键词：{keywords}

任务：
1. 判断这篇论文与现有概念的相关度（0-1）
2. 识别论文中的新概念（不在现有库中的）
3. 识别概念之间的关系

输出 JSON：
{
  "matched_concepts": [
    {
      "concept_name": "AI×心理健康",
      "relevance": 0.92,
      "reason": "论文主要讨论LLM在心理治疗中的应用"
    }
  ],
  "new_concepts": [
    {
      "name": "Prompt Engineering",
      "description": "设计有效的提示词来指导AI模型",
      "keywords": ["prompt", "instruction", "engineering"],
      "parent_concept": "AI×心理健康",
      "relevance": 0.78,
      "context": "论文中详细讨论了如何设计提示词..."
    }
  ],
  "concept_relationships": [
    {
      "source": "AI×心理健康",
      "target": "Prompt Engineering",
      "relation": "uses"
    }
  ]
}
```

---

### Phase 4: 概念审核和合并（用户干预）

**自动识别的新概念状态**：`pending_review`

**用户操作**：

```bash
# 查看待审核概念
/playmore-review-concepts

# 批准概念
/playmore-approve-concept concept_auto_001

# 拒绝概念
/playmore-reject-concept concept_auto_001

# 合并重复概念
/playmore-merge-concepts concept_auto_001 concept_auto_002 --into "Prompt Engineering"

# 编辑概念
/playmore-edit-concept concept_auto_001 --name "Advanced Prompt Engineering"
```

**系统动作**：
1. 更新 concepts.json 中的 status
2. 更新概念笔记
3. 更新 paper_concepts.json 中的映射

---

### Phase 5: 自动整理和层级化

**触发**：定期（如每周）或手动触发

**步骤**：

```
1. 读取所有已批准的概念
2. 调用 Claude 进行层级分析
   - 识别概念的上下位关系
   - 识别概念的聚类
   - 建议概念的重组
3. 生成层级树
4. 更新 concepts.json 中的 parent_concepts / child_concepts
```

**Claude Prompt 示例**：

```
你是一个知识图谱设计专家。

现有概念列表：
- AI×心理健康
- Transformer模型
- Prompt Engineering
- Multimodal Learning
- 深度学习×情绪识别
- CNN架构
- 情绪分类
- 多模态融合

任务：
1. 识别概念的层级关系（parent/child）
2. 识别概念的聚类（相似概念分组）
3. 建议概念的重组或合并

输出 JSON：
{
  "hierarchy": [
    {
      "parent": "AI×心理健康",
      "children": ["Transformer模型", "Prompt Engineering", "Multimodal Learning"]
    },
    {
      "parent": "深度学习×情绪识别",
      "children": ["CNN架构", "情绪分类", "多模态融合"]
    }
  ],
  "clusters": [
    {
      "cluster_name": "模型架构",
      "concepts": ["Transformer模型", "CNN架构"]
    },
    {
      "cluster_name": "应用方向",
      "concepts": ["AI×心理健康", "深度学习×情绪识别"]
    }
  ],
  "merge_suggestions": [
    {
      "concepts": ["情绪分类", "情绪识别"],
      "reason": "这两个概念高度重叠，建议合并为'情绪识别'"
    }
  ]
}
```

---

### Phase 6: 概念库索引生成

**触发**：每次概念更新后

**输出**：

```
concepts/
├── INDEX.md                          # 概念库总索引
├── HIERARCHY.md                      # 层级树
├── concept_001.md                    # AI×心理健康
├── concept_002.md                    # 深度学习×情绪识别
├── concept_auto_001.md               # Transformer模型
└── ...
```

**INDEX.md 示例**：

```markdown
# 概念库索引

## 种子概念（用户定义）
- [AI×心理健康](concept_001.md) - 12篇论文
- [深度学习×情绪识别](concept_002.md) - 8篇论文

## 自动识别概念
### 模型架构
- [Transformer模型](concept_auto_001.md) - 9篇论文
- [CNN架构](concept_auto_003.md) - 5篇论文

### 应用技术
- [Prompt Engineering](concept_auto_002.md) - 7篇论文
- [多模态融合](concept_auto_004.md) - 4篇论文

## 待审核概念
- Reinforcement Learning in Therapy (0.71 confidence)
- Graph Neural Networks (0.65 confidence)

## 最近更新
- 2026-04-28: 新增 Prompt Engineering
- 2026-04-27: 创建种子概念
```

---

## 数据存储结构

```
~/Desktop/playmore/
├── config.json                       # 用户配置 + 种子概念
├── concepts.json                     # 概念库元数据
├── paper_concepts.json               # 论文-概念映射
├── schedule.json                     # 定时任务
├── INDEX.md                          # 论文索引
├── concepts/
│   ├── INDEX.md                      # 概念库总索引
│   ├── HIERARCHY.md                  # 层级树
│   ├── concept_001.md                # 种子概念笔记
│   ├── concept_auto_001.md           # 自动概念笔记
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

## 关键设计决策

### 1. 概念去重和合并

**问题**：AI可能识别出重复或相似的概念

**解决方案**：
- 自动计算概念相似度（基于keywords和description）
- 相似度 > 0.8 时提示用户合并
- 用户可以手动合并

```python
def calculate_concept_similarity(concept1, concept2):
    # 基于keywords的Jaccard相似度
    # 基于description的语义相似度（可用embedding）
    # 返回 0-1 的相似度分数
    pass
```

### 2. 概念的生命周期

```
pending_review → approved → active → archived
                    ↓
                 rejected
```

- **pending_review**：AI自动识别，等待用户审核
- **approved**：用户批准，进入活跃状态
- **active**：正常使用，参与论文分类
- **archived**：用户标记为过时，不再使用
- **rejected**：用户拒绝，不再考虑

### 3. 概念的置信度

每个自动识别的概念都有 confidence score（0-1）：
- 0.9+：高置信度，可能直接批准
- 0.7-0.9：中置信度，需要用户审核
- <0.7：低置信度，可能需要更多证据

### 4. 概念的演进历史

每个概念记录：
- 创建时间
- 最后更新时间
- 状态变化历史
- 相关论文数量变化

这样用户可以看到概念库的演进过程。

---

## 用户交互流程

### 初始化

```bash
# 1. 定义种子概念
cat > ~/Desktop/playmore/config.json << EOF
{
  "seed_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy"]
    },
    {
      "name": "深度学习×情绪识别",
      "keywords": ["deep learning", "emotion", "recognition"]
    }
  ]
}
EOF

# 2. 初始化概念库
/playmore-init-concepts
```

### 日常使用

```bash
# 1. 搜索论文（现有流程）
/playmore AI 心理健康

# 2. 查看待审核概念
/playmore-review-concepts
# 输出：
# ✓ Prompt Engineering (0.78 confidence) - 7篇论文
# ✓ Multimodal Learning (0.72 confidence) - 4篇论文
# ✓ Reinforcement Learning (0.65 confidence) - 2篇论文

# 3. 批准或拒绝
/playmore-approve-concept "Prompt Engineering"
/playmore-reject-concept "Reinforcement Learning"

# 4. 查看概念库
/playmore-concepts list
/playmore-concepts hierarchy
/playmore-concepts search "transformer"

# 5. 定期整理（可选）
/playmore-organize-concepts
```

### 高级操作

```bash
# 合并重复概念
/playmore-merge-concepts "Emotion Recognition" "Emotion Classification"

# 编辑概念
/playmore-edit-concept "Prompt Engineering" \
  --description "设计有效的提示词来指导AI模型" \
  --keywords "prompt,instruction,engineering"

# 重新分类论文
/playmore-reclassify-papers --concept "AI×心理健康"

# 导出概念库
/playmore-export-concepts --format markdown --output concepts_export.md
```

---

## 实现优先级

### Phase 1（必需）
- [ ] 概念库数据结构（concepts.json）
- [ ] 论文-概念映射（paper_concepts.json）
- [ ] 概念笔记生成
- [ ] 基础的概念识别（Claude调用）

### Phase 2（重要）
- [ ] 用户审核界面（/playmore-review-concepts）
- [ ] 概念批准/拒绝逻辑
- [ ] 概念去重检测
- [ ] 概念索引生成

### Phase 3（可选）
- [ ] 自动层级化（/playmore-organize-concepts）
- [ ] 概念合并逻辑
- [ ] 概念编辑界面
- [ ] 概念库导出

---

## 与跨天去重的集成

**跨天去重**（历史管理）和**动态概念库**是独立的两个系统：

- **跨天去重**：避免同一篇论文重复推荐
  - 存储：`schedule.json` 中的 `history` 字段
  - 作用：时间维度的去重

- **动态概念库**：组织和理解论文内容
  - 存储：`concepts.json` + `paper_concepts.json`
  - 作用：知识维度的组织

两者可以协同工作：
- 用户搜索时，系统检查历史（避免重复）
- 新论文进来时，系统自动分类到概念库
- 用户可以按概念浏览论文

