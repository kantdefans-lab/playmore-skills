# 论文对比 & 论文引用 - 深度讨论

## 第一部分：论文对比功能

### 1.1 功能定义

**核心功能**：用户可以选择两篇或多篇论文，系统生成详细的对比分析

**用户场景**：

```
场景1：技术对比
用户想对比两个相似的方法：
- 论文A：使用Transformer的LLM方法
- 论文B：使用CNN的传统方法

系统生成对比：
- 核心算法对比
- 性能指标对比
- 优缺点分析
- 适用场景分析

场景2：时间演进
用户想看某个方法的演进：
- 论文A（2022）：初始方法
- 论文B（2023）：改进版本
- 论文C（2024）：最新版本

系统生成对比：
- 方法的演进过程
- 性能的改进趋势
- 新增的特性

场景3：竞争对比
用户想对比不同团队的方法：
- 论文A：Google的方法
- 论文B：OpenAI的方法
- 论文C：Meta的方法

系统生成对比：
- 技术路线对比
- 性能对比
- 创新点对比
```

---

### 1.2 实现方式

#### 方式A：Claude驱动的对比

```python
def compare_papers(paper_ids, comparison_type="comprehensive"):
    """
    对比多篇论文
    
    comparison_type:
    - "comprehensive": 全面对比
    - "technical": 技术对比
    - "performance": 性能对比
    - "evolution": 演进对比
    """
    
    # Step 1: 获取论文信息
    papers = [get_paper_info(pid) for pid in paper_ids]
    
    # Step 2: 构建对比Prompt
    prompt = build_comparison_prompt(papers, comparison_type)
    
    # Step 3: 调用Claude
    comparison = call_claude(prompt)
    
    # Step 4: 生成对比笔记
    comparison_note = generate_comparison_note(papers, comparison)
    
    # Step 5: 保存
    save_comparison_note(comparison_note)
    
    return comparison_note
```

**Prompt示例**：

```
你是一个学术论文分析专家。

请对比以下论文：

论文1：
- 标题：LLM-based Therapy Assistant
- 作者：Smith, J.
- 年份：2024
- 核心方法：使用Transformer架构的LLM
- 关键结果：85% 准确率
- 摘要：...

论文2：
- 标题：CNN-based Emotion Recognition
- 作者：Johnson, M.
- 年份：2023
- 核心方法：使用CNN的传统方法
- 关键结果：78% 准确率
- 摘要：...

请从以下维度对比：
1. 核心算法和架构
2. 性能指标
3. 优缺点
4. 适用场景
5. 创新点
6. 可能的改进方向

输出格式：结构化的对比分析
```

**Token消耗**：
- 输入：每篇论文 300-400 token
- Prompt：200 token
- 输出：500-800 token
- **总计：1500-2400 token/次**

---

#### 方式B：结构化对比

```python
def structured_comparison(paper_ids):
    """
    结构化对比 - 更高效的方式
    """
    
    papers = [get_paper_info(pid) for pid in paper_ids]
    
    # 提取关键信息（不调用Claude）
    comparison_data = {
        "papers": papers,
        "dimensions": {
            "method": extract_method(papers),
            "performance": extract_performance(papers),
            "dataset": extract_dataset(papers),
            "year": extract_year(papers),
            "authors": extract_authors(papers)
        }
    }
    
    # 生成对比表格
    comparison_table = generate_comparison_table(comparison_data)
    
    # 可选：调用Claude生成分析
    if need_deep_analysis:
        analysis = call_claude_for_analysis(comparison_data)
    
    return comparison_table, analysis
```

**Token消耗**：
- 无Claude调用：0 token
- 可选分析：500-800 token
- **总计：0-800 token/次**

---

### 1.3 用户交互流程

#### 流程1：快速对比

```
用户操作：
1. 打开两篇论文笔记
2. 右键 → "对比这两篇论文"
3. 系统生成对比

输出：
┌─────────────────────────────────────┐
│ 论文对比：LLM vs CNN                 │
├─────────────────────────────────────┤
│                                     │
│ 维度        │ 论文A (LLM)  │ 论文B (CNN)
│ ─────────────┼──────────────┼──────────
│ 年份        │ 2024         │ 2023
│ 方法        │ Transformer  │ CNN
│ 准确率      │ 85%          │ 78%
│ 数据集      │ 1000患者     │ 500患者
│ 优点        │ 更好的性能   │ 更快的推理
│ 缺点        │ 计算量大     │ 性能较低
│ 适用场景    │ 高精度需求   │ 实时应用
│                                     │
└─────────────────────────────────────┘
```

#### 流程2：深度对比

```
用户操作：
1. /playmore-compare paper_001 paper_002 --deep

系统生成：
1. 结构化对比表格
2. Claude深度分析
3. 生成对比笔记

输出文件：
~/Desktop/playmore/comparisons/
  └─ 2024_smith_vs_johnson_llm_vs_cnn.md
```

#### 流程3：多篇对比

```
用户操作：
1. /playmore-compare paper_001 paper_002 paper_003 --evolution

系统生成：
1. 时间线对比
2. 方法演进分析
3. 性能趋势图

输出：
时间线：
2022: 初始方法 (论文A)
  ├─ 准确率：70%
  ├─ 方法：基础CNN
  └─ 数据集：100患者

2023: 改进版本 (论文B)
  ├─ 准确率：78%
  ├─ 方法：改进的CNN
  └─ 数据集：500患者

2024: 最新版本 (论文C)
  ├─ 准确率：85%
  ├─ 方法：Transformer
  └─ 数据集：1000患者
```

---

### 1.4 集成到现有系统

#### 选项1：作为独立功能

```
新增命令：
/playmore-compare paper_001 paper_002

新增文件夹：
~/Desktop/playmore/comparisons/
  └─ comparison_001.md
```

#### 选项2：集成到笔记中

```
在论文笔记中添加"相关论文"section：

## 相关论文
- [[2023_johnson_cnn]] (对比)
- [[2024_williams_multimodal]] (对比)

点击"对比"链接，生成对比笔记
```

#### 选项3：集成到概念库

```
在概念笔记中添加"方法对比"section：

## 方法对比
- Transformer方法 vs CNN方法
- LLM方法 vs 传统方法

点击对比，生成详细分析
```

---

### 1.5 成本分析

#### 成本对比

```
方式A（Claude驱动）：
- 每次对比：1500-2400 token
- 月度（10次）：15000-24000 token
- 成本：$0.06-0.36（Sonnet）

方式B（结构化）：
- 每次对比：0-800 token（可选）
- 月度（10次）：0-8000 token
- 成本：$0-0.12（Sonnet）

节省：50-100%
```

---

### 1.6 你的想法

**问题1**：你认为哪种实现方式更好？
- 方式A（完全由Claude驱动，质量高但成本高）
- 方式B（结构化对比，成本低但需要用户理解）
- 混合方式（简单对比用结构化，复杂对比用Claude）

**问题2**：对比功能应该支持多少篇论文？
- 只支持2篇？
- 支持3-5篇？
- 支持任意数量？

**问题3**：对比的维度应该有多少？
- 基础维度（方法、性能、年份）
- 中等维度（+数据集、优缺点）
- 完整维度（+创新点、适用场景、改进方向）

**问题4**：这个功能应该在MVP还是V2实现？
- MVP（第1-6周）
- V2（第7-12周）

---

## 第二部分：论文引用功能

### 2.1 功能定义

**核心功能**：显示论文之间的引用关系，构建引用网络

**用户场景**：

```
场景1：追踪论文影响力
用户想了解一篇论文的影响：
- 这篇论文被引用了多少次？
- 哪些论文引用了它？
- 它的影响力如何？

场景2：发现关键论文
用户想找到领域的关键论文：
- 哪些论文被引用最多？
- 哪些论文是基础性的？
- 哪些论文是最新的？

场景3：理解研究脉络
用户想理解研究的发展过程：
- 这个领域是如何发展的？
- 哪些论文是里程碑？
- 研究方向如何演进？

场景4：发现相关工作
用户想找到相关的论文：
- 这篇论文引用了哪些论文？
- 哪些论文与这篇论文相关？
- 如何找到完整的相关工作？
```

---

### 2.2 数据来源

#### 来源1：API获取

```python
def fetch_citations(paper_doi):
    """从API获取引用信息"""
    
    # 来源1：Semantic Scholar API
    semantic_scholar_citations = call_semantic_scholar_api(paper_doi)
    
    # 来源2：OpenAlex API
    openalex_citations = call_openalex_api(paper_doi)
    
    # 来源3：CrossRef API
    crossref_citations = call_crossref_api(paper_doi)
    
    # 合并和去重
    all_citations = merge_and_deduplicate(
        semantic_scholar_citations,
        openalex_citations,
        crossref_citations
    )
    
    return all_citations
```

**Token消耗**：
- 0 token（只调用外部API）

#### 来源2：用户论文库

```python
def find_citations_in_library(paper_id):
    """在用户的论文库中查找引用关系"""
    
    # 获取论文信息
    paper = get_paper_info(paper_id)
    
    # 在所有论文中搜索
    citing_papers = []
    for other_paper in all_papers:
        if paper_id in other_paper.get("references", []):
            citing_papers.append(other_paper)
    
    return citing_papers
```

**Token消耗**：
- 0 token（本地搜索）

---

### 2.3 可视化方式

#### 方式1：引用树

```
论文A（2024）
├─ 引用了：
│  ├─ 论文B（2023）
│  │  ├─ 引用了：论文C（2022）
│  │  └─ 引用了：论文D（2022）
│  └─ 论文E（2023）
│     └─ 引用了：论文F（2021）
│
└─ 被引用了：
   ├─ 论文G（2024）
   └─ 论文H（2024）
```

#### 方式2：引用网络图

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

#### 方式3：引用统计

```
论文A的引用统计：

被引用次数：45次
  ├─ 2024年：15次
  ├─ 2023年：20次
  └─ 2022年：10次

引用的论文数：12篇
  ├─ 2023年：5篇
  ├─ 2022年：4篇
  └─ 2021年：3篇

影响力排名：领域前10%
```

---

### 2.4 实现方式

#### 方式A：完整引用网络

```python
def build_citation_network(paper_id, depth=2):
    """
    构建完整的引用网络
    
    depth: 网络深度
    - 1: 只显示直接引用
    - 2: 显示引用的引用
    - 3+: 更深的网络
    """
    
    # Step 1: 获取论文的引用
    citations = fetch_citations(paper_id)
    
    # Step 2: 递归获取引用的引用
    network = {
        "paper": paper_id,
        "cites": [],
        "cited_by": []
    }
    
    for citation in citations:
        if depth > 1:
            sub_network = build_citation_network(citation["id"], depth-1)
            network["cites"].append(sub_network)
    
    # Step 3: 获取被引用的论文
    citing_papers = fetch_citing_papers(paper_id)
    for citing in citing_papers:
        network["cited_by"].append(citing)
    
    return network
```

**Token消耗**：
- 0 token（只调用外部API）

#### 方式B：简化版本

```python
def simple_citation_view(paper_id):
    """
    简化的引用视图
    """
    
    # 只显示直接引用
    citations = fetch_citations(paper_id)
    citing_papers = fetch_citing_papers(paper_id)
    
    return {
        "cites": citations[:10],  # 只显示前10个
        "cited_by": citing_papers[:10]
    }
```

**Token消耗**：
- 0 token

---

### 2.5 用户交互流程

#### 流程1：查看论文的引用

```
用户操作：
1. 打开论文笔记
2. 点击"引用关系"tab
3. 系统显示引用信息

输出：
┌─────────────────────────────────────┐
│ 论文A的引用关系                      │
├─────────────────────────────────────┤
│                                     │
│ 📊 统计信息                          │
│ ├─ 被引用次数：45次                  │
│ ├─ 引用的论文：12篇                  │
│ └─ 影响力排名：前10%                 │
│                                     │
│ 📖 这篇论文引用了：                  │
│ ├─ 论文B (2023) - 被引用100次        │
│ ├─ 论文C (2022) - 被引用80次         │
│ └─ ... (更多)                       │
│                                     │
│ 📚 引用这篇论文的：                  │
│ ├─ 论文D (2024) - 新论文             │
│ ├─ 论文E (2024) - 新论文             │
│ └─ ... (更多)                       │
│                                     │
└─────────────────────────────────────┘
```

#### 流程2：探索引用网络

```
用户操作：
1. /playmore-citations network paper_001 --depth 2

系统生成：
1. 引用网络图
2. 网络统计
3. 关键论文识别

输出：
- 可视化的引用网络
- 关键论文列表
- 研究脉络分析
```

#### 流程3：发现关键论文

```
用户操作：
1. /playmore-citations top-cited --concept "AI×心理健康"

系统生成：
1. 该概念下被引用最多的论文
2. 排序和统计
3. 推荐阅读

输出：
🔥 该领域最有影响力的论文：

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

---

### 2.6 集成到现有系统

#### 选项1：论文笔记中的引用tab

```
论文笔记中添加新的tab：
- 📄 内容
- 🏷️ 概念
- 📊 引用关系 ← 新增
- 💬 讨论
```

#### 选项2：概念笔记中的引用分析

```
概念笔记中添加：

## 关键论文
- 论文A (被引用500次)
- 论文B (被引用400次)
- 论文C (被引用300次)

## 研究脉络
[可视化的引用网络]
```

#### 选项3：独立的引用浏览器

```
新增命令：
/playmore-citations browse

打开交互式的引用浏览器
```

---

### 2.7 成本分析

#### 成本对比

```
API调用成本：
- Semantic Scholar API：免费
- OpenAlex API：免费
- CrossRef API：免费

Claude调用成本：
- 基础版本（无Claude）：0 token
- 分析版本（可选Claude）：500-1000 token

总成本：
- 基础版本：$0
- 分析版本：$0.02-0.15（Sonnet）
```

---

### 2.8 你的想法

**问题1**：引用功能应该支持多深的网络？
- 只显示直接引用（depth=1）
- 显示引用的引用（depth=2）
- 更深的网络（depth=3+）

**问题2**：应该如何处理引用数据的更新？
- 每次查看时实时获取（最新但慢）
- 定期缓存（快但可能过期）
- 用户手动刷新

**问题3**：引用功能应该支持哪些操作？
- 只查看（基础）
- 查看+过滤（中等）
- 查看+过滤+可视化+分析（完整）

**问题4**：这个功能应该在MVP还是V2实现？
- MVP（第1-6周）
- V2（第7-12周）

---

## 第三部分：两个功能的对比

### 对比表

| 维度 | 论文对比 | 论文引用 |
|------|---------|---------|
| **用户价值** | 高 | 高 |
| **实现复杂度** | 中 | 低 |
| **Token成本** | 高 | 低 |
| **数据来源** | 论文库 | API |
| **实时性** | 高 | 中 |
| **用户频率** | 中 | 中 |
| **优先级** | 中 | 高 |

---

### 优先级建议

**我的建议**：
1. **先实现论文引用**（V2早期）
   - 成本低（0 token）
   - 实现简单
   - 用户价值高
   - 可以快速上线

2. **后实现论文对比**（V2中期）
   - 成本高（1500-2400 token）
   - 实现复杂
   - 用户价值高
   - 需要更多规划

---

## 现在让我们讨论

### 关键问题

1. **你认为这两个功能哪个更重要？**
   - 论文对比？
   - 论文引用？
   - 一样重要？

2. **对于论文对比，你更倾向哪种实现方式？**
   - 方式A（Claude驱动，质量高）
   - 方式B（结构化，成本低）
   - 混合方式？

3. **对于论文引用，你认为应该支持多深的网络？**
   - depth=1（只显示直接引用）
   - depth=2（显示引用的引用）
   - depth=3+（更深的网络）

4. **这两个功能应该什么时候实现？**
   - 都在MVP中？
   - 都在V2中？
   - 一个MVP一个V2？

5. **还有其他想法或改进吗？**
   - 这两个功能还可以怎么优化？
   - 是否应该结合使用？
   - 是否有其他相关功能？

