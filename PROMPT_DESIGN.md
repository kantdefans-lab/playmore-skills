# Claude Prompt设计 - 详细分析

## 问题1：如何不遗漏重要细节？

### 方案对比

#### 方案A：一次处理多篇论文

```
输入：5篇论文的标题+摘要
输出：所有论文的概念识别结果
```

**优点**：
- ✅ Token效率高（一次调用处理多篇）
- ✅ 可以识别跨论文的概念关系
- ✅ 速度快

**缺点**：
- ❌ Context可能过长
- ❌ 容易遗漏细节
- ❌ 难以调试（哪篇论文出问题？）
- ❌ 如果某篇论文很复杂，可能被忽视

**遗漏的风险**：
```
论文1：简单的LLM应用
论文2：复杂的多模态融合 ← 可能被忽视
论文3：简单的情绪识别
论文4：简单的数据集
论文5：简单的基准测试

Claude可能重点关注论文1/3/4/5，忽视论文2的复杂细节
```

---

#### 方案B：逐篇处理

```
输入：1篇论文的标题+摘要
输出：该论文的概念识别结果

重复5次
```

**优点**：
- ✅ 精准度高（每篇论文都得到充分关注）
- ✅ 易于调试
- ✅ 不会遗漏细节
- ✅ 可以针对复杂论文调整prompt

**缺点**：
- ❌ Token成本高（5次调用）
- ❌ 速度慢
- ❌ 无法识别跨论文的概念关系

---

#### 方案C：混合方案（推荐）

**核心思想**：根据论文复杂度动态调整

```
Step 1: 快速扫描所有论文
  - 计算每篇论文的"复杂度"（摘要长度、关键词数量等）
  - 分类为"简单"和"复杂"

Step 2: 分别处理
  - 简单论文：批量处理（5篇一组）
  - 复杂论文：逐篇处理

Step 3: 识别跨论文关系
  - 在最后一步识别概念之间的关系
```

**示例**：
```
论文1（简单）+ 论文3（简单）+ 论文4（简单）+ 论文5（简单）
  → 一次调用处理

论文2（复杂）
  → 单独调用处理

最后：识别所有概念的关系
```

**优点**：
- ✅ 精准度高（复杂论文不会被忽视）
- ✅ Token效率相对高（简单论文批量处理）
- ✅ 速度相对快
- ✅ 易于调试

**缺点**：
- ⚠️ 实现稍微复杂一点

---

### 我的建议：**方案C（混合方案）**

**具体实现**：

```python
def calculate_complexity(paper):
    """计算论文复杂度"""
    abstract_length = len(paper.get("abstract", ""))
    keywords_count = len(paper.get("keywords", []))
    title_length = len(paper.get("title", ""))
    
    # 简单的复杂度评分
    complexity_score = (
        (abstract_length / 500) * 0.5 +  # 摘要长度
        (keywords_count / 10) * 0.3 +    # 关键词数量
        (title_length / 100) * 0.2       # 标题长度
    )
    
    return complexity_score

def group_papers(papers):
    """根据复杂度分组"""
    simple_papers = []
    complex_papers = []
    
    for paper in papers:
        if calculate_complexity(paper) > 0.6:
            complex_papers.append(paper)
        else:
            simple_papers.append(paper)
    
    return simple_papers, complex_papers

def process_papers(papers):
    """处理论文"""
    simple_papers, complex_papers = group_papers(papers)
    
    all_concepts = []
    
    # 处理简单论文（批量）
    if simple_papers:
        batch_size = 5
        for i in range(0, len(simple_papers), batch_size):
            batch = simple_papers[i:i+batch_size]
            concepts = call_claude_batch(batch)
            all_concepts.extend(concepts)
    
    # 处理复杂论文（逐篇）
    for paper in complex_papers:
        concepts = call_claude_single(paper)
        all_concepts.extend(concepts)
    
    # 识别跨论文关系
    relationships = identify_relationships(all_concepts)
    
    return all_concepts, relationships
```

---

## 问题2：具体的Prompt是什么样的？

### Prompt 1：批量处理简单论文

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
    "id": "paper_001",
    "title": "LLM-based Therapy Assistant",
    "abstract": "We propose an LLM-based system for mental health support...",
    "keywords": ["LLM", "therapy", "mental health"]
  },
  {
    "id": "paper_003",
    "title": "Emotion Recognition from Text",
    "abstract": "We develop a deep learning model for emotion recognition...",
    "keywords": ["emotion", "recognition", "deep learning"]
  },
  {
    "id": "paper_004",
    "title": "A New Dataset for Sentiment Analysis",
    "abstract": "We introduce a large-scale dataset for sentiment analysis...",
    "keywords": ["dataset", "sentiment", "analysis"]
  }
]

任务：
对每篇论文，执行以下操作：

1. 判断与现有概念的相关度（0-1）
   - 检查标题、摘要、关键词中是否出现概念的关键词
   - 计算相关度分数

2. 识别新概念（不在现有库中的）
   - 提取论文中的关键技术、方法、应用领域
   - 排除通用词、人名、地名
   - 为每个新概念计算置信度（0-1）

3. 识别概念关系
   - 新概念与现有概念的关系（parent/related/similar）

输出 JSON 数组（每篇论文一个对象）：
[
  {
    "paper_id": "paper_001",
    "matched_concepts": [
      {
        "concept_name": "AI×心理健康",
        "relevance": 0.95,
        "reason": "论文标题和摘要都明确提到LLM用于心理健康支持"
      }
    ],
    "new_concepts": [
      {
        "name": "Prompt Engineering",
        "description": "设计有效的提示词来指导LLM的行为",
        "keywords": ["prompt", "instruction", "engineering"],
        "parent_concept": "AI×心理健康",
        "relevance": 0.78,
        "confidence": 0.85,
        "context": "论文中讨论了如何设计提示词来改进治疗效果"
      }
    ]
  },
  {
    "paper_id": "paper_003",
    "matched_concepts": [
      {
        "concept_name": "深度学习×情绪识别",
        "relevance": 0.92,
        "reason": "论文直接关于情绪识别的深度学习方法"
      }
    ],
    "new_concepts": []
  },
  {
    "paper_id": "paper_004",
    "matched_concepts": [],
    "new_concepts": [
      {
        "name": "Sentiment Analysis Dataset",
        "description": "用于情感分析的大规模标注数据集",
        "keywords": ["dataset", "sentiment", "annotation"],
        "parent_concept": "深度学习×情绪识别",
        "relevance": 0.65,
        "confidence": 0.72,
        "context": "论文介绍了一个新的情感分析数据集"
      }
    ]
  }
]

重要提示：
- 每篇论文必须单独分析，不要混淆
- 置信度基于：关键词匹配度 + 在多篇论文中出现的频率 + 你的评估
- 不要凭空说"只有仿真"或"山寨"，除非有具体证据
- 不确定的信息必须注明"摘要未提及"
```

---

### Prompt 2：逐篇处理复杂论文

```
你是一个学术概念分类专家。

已有的概念库：
{
  "seed_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy", "心理", "治疗"]
    }
  ],
  "auto_concepts": [
    {
      "name": "Transformer模型",
      "keywords": ["transformer", "attention", "BERT"]
    },
    {
      "name": "Prompt Engineering",
      "keywords": ["prompt", "instruction", "engineering"]
    }
  ]
}

论文详细信息：
{
  "id": "paper_002",
  "title": "Multimodal Learning for Mental Health Assessment: Integrating Text, Speech, and Physiological Signals",
  "abstract": "Mental health assessment is crucial for early intervention. We propose a novel multimodal learning framework that integrates text-based conversations, speech analysis, and physiological signals (heart rate, skin conductance) to provide comprehensive mental health assessment. Our approach uses transformer-based architectures for feature extraction and fusion. We evaluate on a new dataset of 1000 patients with clinical annotations. Results show 85% accuracy in depression detection and 78% in anxiety detection.",
  "keywords": ["multimodal learning", "mental health", "transformer", "physiological signals", "depression detection"],
  "authors": ["Smith, J.", "Johnson, M.", "Williams, K."],
  "year": 2024,
  "venue": "Nature Mental Health"
}

任务：
这是一篇复杂的论文，需要深入分析。请执行以下操作：

1. 详细分析论文内容
   - 核心贡献是什么？
   - 使用了哪些技术？
   - 应用领域是什么？

2. 匹配现有概念
   - 与"AI×心理健康"的相关度？
   - 与"Transformer模型"的相关度？
   - 与"Prompt Engineering"的相关度？
   - 是否还有其他相关的现有概念？

3. 识别新概念
   - 这篇论文引入了哪些新的概念或方法？
   - 例如：Multimodal Learning, Physiological Signal Processing, etc.
   - 为每个新概念计算置信度

4. 识别概念关系
   - 新概念与现有概念的关系
   - 新概念之间的关系

输出 JSON 对象：
{
  "paper_id": "paper_002",
  "analysis": {
    "core_contribution": "提出了一个多模态学习框架，整合文本、语音和生理信号用于心理健康评估",
    "technologies_used": ["transformer", "multimodal fusion", "physiological signal processing"],
    "application_domain": "mental health assessment"
  },
  "matched_concepts": [
    {
      "concept_name": "AI×心理健康",
      "relevance": 0.98,
      "reason": "论文直接关于AI在心理健康评估中的应用，包括抑郁症和焦虑症检测"
    },
    {
      "concept_name": "Transformer模型",
      "relevance": 0.85,
      "reason": "论文使用transformer架构进行特征提取和融合"
    }
  ],
  "new_concepts": [
    {
      "name": "Multimodal Learning",
      "description": "融合多种模态（文本、语音、生理信号等）的学习方法",
      "keywords": ["multimodal", "fusion", "cross-modal", "multi-modal"],
      "parent_concept": "AI×心理健康",
      "relevance": 0.92,
      "confidence": 0.95,
      "context": "论文的核心方法是整合文本、语音和生理信号的多模态学习框架"
    },
    {
      "name": "Physiological Signal Processing",
      "description": "处理和分析生理信号（心率、皮肤电导等）的技术",
      "keywords": ["physiological", "signal processing", "heart rate", "skin conductance"],
      "parent_concept": "AI×心理健康",
      "relevance": 0.88,
      "confidence": 0.90,
      "context": "论文使用生理信号作为心理健康评估的重要输入"
    },
    {
      "name": "Depression Detection",
      "description": "使用机器学习方法自动检测抑郁症的技术",
      "keywords": ["depression", "detection", "classification"],
      "parent_concept": "AI×心理健康",
      "relevance": 0.85,
      "confidence": 0.88,
      "context": "论文在抑郁症检测上达到85%的准确率"
    }
  ],
  "concept_relationships": [
    {
      "source": "Multimodal Learning",
      "target": "Transformer模型",
      "relation": "uses",
      "strength": 0.9
    },
    {
      "source": "Multimodal Learning",
      "target": "Physiological Signal Processing",
      "relation": "related_to",
      "strength": 0.85
    }
  ]
}

重要提示：
- 这是一篇复杂的论文，需要深入理解
- 不要遗漏任何重要的技术或方法
- 置信度应该基于论文中的具体证据
- 如果论文中没有提到某个信息，请注明"论文未提及"
```

---

### Prompt 3：识别跨论文的概念关系

```
你是一个知识图谱设计专家。

已识别的所有概念：
[
  {
    "name": "AI×心理健康",
    "type": "seed",
    "paper_count": 5
  },
  {
    "name": "Transformer模型",
    "type": "auto",
    "paper_count": 8
  },
  {
    "name": "Prompt Engineering",
    "type": "auto",
    "paper_count": 3
  },
  {
    "name": "Multimodal Learning",
    "type": "auto",
    "paper_count": 4
  },
  {
    "name": "Physiological Signal Processing",
    "type": "auto",
    "paper_count": 2
  },
  {
    "name": "Depression Detection",
    "type": "auto",
    "paper_count": 2
  }
]

论文-概念映射：
{
  "paper_001": ["AI×心理健康", "Transformer模型", "Prompt Engineering"],
  "paper_002": ["AI×心理健康", "Transformer模型", "Multimodal Learning", "Physiological Signal Processing", "Depression Detection"],
  "paper_003": ["AI×心理健康", "Depression Detection"],
  ...
}

任务：
识别概念之间的层级和关系：

1. 识别parent/child关系
   - 哪些概念应该是其他概念的子概念？
   - 例如：Depression Detection 是 AI×心理健康 的子概念

2. 识别related关系
   - 哪些概念相关但不是parent/child关系？
   - 例如：Multimodal Learning 和 Physiological Signal Processing 相关

3. 识别similar关系
   - 哪些概念应该合并？
   - 例如：如果有"Emotion Recognition"和"Emotion Detection"，应该合并

输出 JSON：
{
  "hierarchy": [
    {
      "parent": "AI×心理健康",
      "children": [
        "Transformer模型",
        "Prompt Engineering",
        "Multimodal Learning",
        "Physiological Signal Processing",
        "Depression Detection"
      ]
    },
    {
      "parent": "Multimodal Learning",
      "children": [
        "Physiological Signal Processing"
      ]
    }
  ],
  "relationships": [
    {
      "source": "Transformer模型",
      "target": "Multimodal Learning",
      "relation": "used_by",
      "strength": 0.9
    },
    {
      "source": "Prompt Engineering",
      "target": "AI×心理健康",
      "relation": "technique_for",
      "strength": 0.8
    }
  ],
  "merge_suggestions": [
    {
      "concepts": ["Emotion Recognition", "Emotion Detection"],
      "reason": "这两个概念高度重叠，建议合并为'Emotion Recognition'"
    }
  ]
}

重要提示：
- 基于论文中的共现频率和语义关系
- 不要过度分类，保持层级简洁
- 如果不确定，保持概念独立
```

---

## 总结

### 三个Prompt的用途

| Prompt | 用途 | 输入 | 输出 |
|--------|------|------|------|
| **Prompt 1** | 批量处理简单论文 | 5篇简单论文 | 每篇的概念识别结果 |
| **Prompt 2** | 逐篇处理复杂论文 | 1篇复杂论文 | 该论文的详细概念分析 |
| **Prompt 3** | 识别跨论文关系 | 所有概念 + 论文映射 | 概念的层级和关系 |

### 如何不遗漏细节

1. **复杂度分析** - 识别复杂论文，单独处理
2. **逐篇处理** - 复杂论文不会被忽视
3. **详细Prompt** - 为复杂论文提供更详细的指导
4. **跨论文分析** - 最后识别概念之间的关系

### Token成本估算

```
假设有10篇论文：
- 8篇简单论文：2次调用（4篇/次）
- 2篇复杂论文：2次调用（1篇/次）
- 1次关系识别：1次调用

总计：5次调用

vs 逐篇处理：10次调用

节省：50% token
```

