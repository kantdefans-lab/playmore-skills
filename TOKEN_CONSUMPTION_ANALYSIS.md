# 论文处理的Token消耗点分析

## 关键问题

**处理论文的token消耗点在哪里？是直接把PDF丢给AI处理吗？**

答案：**不是！** 是把结构化数据（标题、摘要、关键词）发给Claude，不是PDF。

---

## 完整的数据流

### 当前的playmore流程

```
用户输入：/playmore AI 心理健康
    ↓
playmore-fetch
    ├─ OpenAlex API → 获取结构化数据
    ├─ Semantic Scholar API → 获取结构化数据
    ├─ Google Scholar → 获取结构化数据
    └─ CNKI → 获取结构化数据
    ↓
scored_papers.json（结构化数据）
    {
      "title": "...",
      "abstract": "...",
      "keywords": [...],
      "authors": [...],
      "year": 2024,
      "journal": "...",
      "doi": "...",
      "pdf_url": "...",
      "sources": [...]
    }
    ↓
playmore-score（打分）
    ↓
playmore-review（AI点评）
    ↓
playmore-concepts-extract（新增：概念识别）← 这里消耗token
```

---

## Token消耗点详细分析

### 1️⃣ playmore-fetch（不消耗Claude token）

```
调用外部API：
- OpenAlex API
- Semantic Scholar API
- Google Scholar（通过gs-skills）
- CNKI（通过cnki-skills）

返回结构化数据：
{
  "title": "LLM-based Therapy Assistant",
  "abstract": "We propose an LLM-based system...",
  "keywords": ["LLM", "therapy", "mental health"],
  "authors": ["Smith, J.", "Johnson, M."],
  "year": 2024,
  "journal": "Nature Mental Health",
  "doi": "10.1234/example",
  "pdf_url": "https://...",
  "cited_by": 15,
  "sources": ["OpenAlex", "Google Scholar"]
}
```

**Token消耗**：
- ❌ **0 Claude token**（只调用外部API）

---

### 2️⃣ playmore-score（不消耗Claude token）

```
输入：scored_papers.json

处理：
- 读取config.json中的keywords
- 计算关键词匹配分数
- 计算期刊分区分数
- 计算Q3+加权分数

输出：
{
  "title": "...",
  "score": 8,
  "score_breakdown": {
    "keyword": 3,
    "tier": 2,
    "boost": 2
  }
}
```

**Token消耗**：
- ❌ **0 Claude token**（纯Python计算）

---

### 3️⃣ playmore-review（消耗Claude token）

```
输入：scored_papers.json（打分后）

Claude Prompt：
"以毒舌但有料的资深研究员角色点评这些论文..."

处理每篇论文：
- 标题：50 token
- 摘要：200-300 token
- 关键词：50 token
- 点评生成：200-300 token

总计：500-700 token/篇

输出：
{
  "title": "...",
  "review": "🔥必读",
  "commentary": "..."
}
```

**Token消耗**：
- ✅ **500-700 token/篇**（Claude调用）
- 10篇论文：5000-7000 token

---

### 4️⃣ playmore-concepts-extract（新增，消耗Claude token）

```
输入：scored_papers.json（打分和点评后）

Step 1: 计算复杂度（0 token）
  - 纯Python计算
  - 分类为simple/complex

Step 2: 批量处理简单论文（消耗token）
  Claude Prompt：
  "识别这5篇论文中的概念..."
  
  输入数据/篇：
  - 标题：50 token
  - 摘要：200-300 token
  - 关键词：50 token
  - 已有概念库：100 token
  
  总计：400-500 token/5篇 = 80-100 token/篇

Step 3: 逐篇处理复杂论文（消耗token）
  Claude Prompt：
  "深入分析这篇复杂论文..."
  
  输入数据/篇：
  - 标题：50 token
  - 摘要：300-400 token
  - 关键词：50 token
  - 已有概念库：100 token
  - 详细分析指导：200 token
  
  总计：700-800 token/篇

Step 4: 识别跨论文关系（消耗token）
  Claude Prompt：
  "识别这些概念的层级关系..."
  
  输入数据：
  - 所有概念列表：200 token
  - 论文-概念映射：300 token
  - 分析指导：100 token
  
  总计：600 token
```

**Token消耗**：
- 简单论文（8篇）：80-100 token/篇 × 8 = 640-800 token
- 复杂论文（2篇）：700-800 token/篇 × 2 = 1400-1600 token
- 关系识别：600 token
- **总计：2640-3000 token**

---

## 完整的Token消耗分布

### 10篇论文的完整处理

```
playmore-fetch
  → 0 token（外部API）

playmore-score
  → 0 token（Python计算）

playmore-review
  → 5000-7000 token（Claude点评）

playmore-concepts-extract（新增）
  → 2640-3000 token（Claude概念识别）

playmore-notes
  → 2000-3000 token（Claude生成笔记）

总计：9640-13000 token
```

---

## 关键点：不是处理PDF

### ❌ 错误的方式（消耗大量token）

```
直接把PDF发给Claude：
- PDF转文本：可能5000-10000 token
- Claude处理：5000-10000 token
- 总计：10000-20000 token/篇

10篇论文：100000-200000 token ❌ 太贵！
```

### ✅ 正确的方式（我们的方式）

```
只发送结构化数据：
- 标题：50 token
- 摘要：200-300 token
- 关键词：50 token
- 总计：300-400 token/篇

10篇论文：3000-4000 token ✅ 高效！
```

---

## 数据流图

```
用户输入
    ↓
playmore-fetch
    ├─ OpenAlex API
    ├─ Semantic Scholar API
    ├─ Google Scholar
    └─ CNKI
    ↓
结构化数据（JSON）
    {
      "title": "...",
      "abstract": "...",
      "keywords": [...],
      "authors": [...],
      "year": 2024,
      "journal": "...",
      "doi": "...",
      "pdf_url": "..."
    }
    ↓
playmore-score（Python计算）
    ↓
playmore-review（Claude处理）
    ├─ 输入：标题 + 摘要 + 关键词
    ├─ 消耗：500-700 token/篇
    └─ 输出：点评
    ↓
playmore-concepts-extract（Claude处理）← 新增
    ├─ 输入：标题 + 摘要 + 关键词 + 已有概念库
    ├─ 消耗：80-800 token/篇（取决于复杂度）
    └─ 输出：识别的概念
    ↓
playmore-notes（Claude处理）
    ├─ 输入：标题 + 摘要 + 关键词 + 点评 + 概念
    ├─ 消耗：200-300 token/篇
    └─ 输出：结构化笔记
    ↓
最终输出：论文笔记 + 概念库
```

---

## 关键的优化点

### 1. 不处理PDF

```
❌ 不做：
  PDF → 文本提取 → Claude处理

✅ 做：
  API → 结构化数据 → Claude处理
```

### 2. 只发送必要的信息

```
❌ 不做：
  发送完整的论文内容（太长）

✅ 做：
  只发送：标题 + 摘要 + 关键词
```

### 3. 根据复杂度调整

```
❌ 不做：
  所有论文都逐篇处理

✅ 做：
  简单论文批量处理
  复杂论文逐篇处理
```

### 4. 缓存已有概念

```
❌ 不做：
  每次都重新识别所有概念

✅ 做：
  缓存已识别的概念
  只识别新概念
```

---

## 具体的Prompt示例

### 发送给Claude的实际数据

```json
{
  "papers": [
    {
      "id": "paper_001",
      "title": "LLM-based Therapy Assistant",
      "abstract": "We propose an LLM-based system for mental health support. The system uses large language models to provide personalized therapy recommendations based on user conversations. We evaluate the system on a dataset of 500 patients and show 85% satisfaction rate.",
      "keywords": ["LLM", "therapy", "mental health", "NLP"]
    },
    {
      "id": "paper_002",
      "title": "Emotion Recognition from Text",
      "abstract": "We develop a deep learning model for emotion recognition from text. The model uses transformer architecture and achieves 92% accuracy on benchmark datasets.",
      "keywords": ["emotion", "recognition", "deep learning", "transformer"]
    }
  ],
  "existing_concepts": [
    {
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy"]
    }
  ]
}
```

**这个JSON的token数**：
- 标题：50 token
- 摘要：200-300 token
- 关键词：50 token
- 已有概念：50 token
- **总计：350-450 token**

---

## Token消耗的三个主要来源

### 1. 输入数据（必需）

```
标题 + 摘要 + 关键词 + 已有概念库
= 300-400 token/篇
```

### 2. Prompt指导（必需）

```
"你是一个学术概念分类专家..."
"任务：识别概念..."
"输出格式：JSON..."
= 200-300 token
```

### 3. 输出结果（必需）

```
Claude生成的概念识别结果
= 200-400 token
```

**总计**：700-1100 token/篇（逐篇处理）

---

## 优化建议

### 1. 使用缓存

```python
# 缓存已识别的概念
cache = {
    "paper_001": {
        "concepts": [...],
        "timestamp": "2026-04-27"
    }
}

# 重复处理同一篇论文时，直接返回缓存
if paper_id in cache:
    return cache[paper_id]
```

### 2. 批量处理

```python
# 简单论文5篇一组
batch_size = 5
for i in range(0, len(simple_papers), batch_size):
    batch = simple_papers[i:i+batch_size]
    # 一次调用处理5篇
    concepts = call_claude_batch(batch)
```

### 3. 压缩数据

```python
# 只发送必要的字段
minimal_paper = {
    "id": paper["id"],
    "title": paper["title"],
    "abstract": paper["abstract"][:500],  # 限制长度
    "keywords": paper["keywords"][:10]    # 限制数量
}
```

---

## 最终的Token成本估算

### 处理10篇论文的完整流程

```
playmore-fetch：0 token
playmore-score：0 token
playmore-review：5000-7000 token
playmore-concepts-extract：2640-3000 token
playmore-notes：2000-3000 token

总计：9640-13000 token

成本（按Claude Haiku）：
- 输入：$0.80/百万token
- 输出：$4/百万token
- 估算成本：$0.10-0.15

成本（按Claude Sonnet）：
- 输入：$3/百万token
- 输出：$15/百万token
- 估算成本：$0.40-0.60
```

---

## 总结

### ✅ 我们的方式

```
API → 结构化数据 → Claude处理
  ↓
只发送：标题 + 摘要 + 关键词
  ↓
Token消耗：300-400 token/篇
  ↓
成本：低 ✅
```

### ❌ 错误的方式

```
PDF → 文本提取 → Claude处理
  ↓
发送：完整论文内容
  ↓
Token消耗：5000-10000 token/篇
  ↓
成本：高 ❌
```

