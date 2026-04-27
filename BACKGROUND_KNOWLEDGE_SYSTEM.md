# 智能背景知识系统 - 详细设计

## 第一部分：整体架构

### 系统流程

```
用户搜索：/playmore AI 心理健康
    ↓
playmore-fetch（获取论文）
    ↓
playmore-score（打分排序）
    ↓
[新增] 背景知识构建系统
    ├─ Step 1: 识别领域和关键概念
    ├─ Step 2: 查找领域大牛和高分区文章
    ├─ Step 3: 查找领域大综述
    ├─ Step 4: 缓存背景知识
    └─ Step 5: 生成背景知识摘要
    ↓
playmore-review（AI锐评，使用背景知识）
    ↓
playmore-concepts-extract（概念识别，使用背景知识）
    ↓
playmore-notes（生成笔记）
```

---

## 第二部分：背景知识构建系统

### 2.1 识别领域和关键概念

```python
def identify_domain_and_concepts(keywords, seed_concepts):
    """
    识别搜索的领域和关键概念
    """
    
    # Step 1: 分析关键词
    analysis = {
        "keywords": keywords,
        "seed_concepts": seed_concepts,
        "domain": identify_domain(keywords, seed_concepts),
        "sub_domains": identify_sub_domains(keywords),
        "key_concepts": extract_key_concepts(keywords)
    }
    
    return analysis

# 示例
keywords = "AI 心理健康"
seed_concepts = ["AI×心理健康"]

result = identify_domain_and_concepts(keywords, seed_concepts)
# 输出：
# {
#   "domain": "AI in Mental Health",
#   "sub_domains": ["LLM applications", "Emotion recognition", "Multimodal learning"],
#   "key_concepts": ["LLM", "Transformer", "Multimodal", "Mental health assessment"]
# }
```

---

### 2.2 查找领域大牛和高分区文章

#### 方案A：使用OpenAlex API

```python
def find_domain_experts_and_papers(domain, sub_domains, top_n=5):
    """
    查找领域大牛和他们的高分区文章
    """
    
    experts = []
    
    # Step 1: 查找该领域的顶级作者
    for sub_domain in sub_domains:
        # 使用OpenAlex API查找该子领域的高被引作者
        authors = query_openalex_top_authors(
            query=sub_domain,
            sort="cited_by_count:desc",
            limit=top_n
        )
        
        for author in authors:
            # Step 2: 获取该作者的高分区文章
            papers = query_openalex_author_papers(
                author_id=author["id"],
                filters={
                    "publication_year": ">2020",  # 最近5年
                    "is_oa": True  # 开放获取
                },
                sort="cited_by_count:desc",
                limit=3  # 每个作者取3篇
            )
            
            experts.append({
                "author": author["name"],
                "affiliation": author.get("affiliation"),
                "cited_by_count": author["cited_by_count"],
                "papers": papers
            })
    
    return experts

# 示例
experts = find_domain_experts_and_papers(
    domain="AI in Mental Health",
    sub_domains=["LLM applications", "Emotion recognition"],
    top_n=3
)

# 输出：
# [
#   {
#     "author": "Smith, J.",
#     "affiliation": "Stanford University",
#     "cited_by_count": 5000,
#     "papers": [
#       {
#         "title": "LLM-based Therapy Assistant",
#         "year": 2024,
#         "cited_by": 150,
#         "journal": "Nature Mental Health"
#       },
#       ...
#     ]
#   },
#   ...
# ]
```

**Token消耗**：0 token（只调用API）

---

#### 方案B：使用Semantic Scholar API

```python
def find_experts_semantic_scholar(domain, top_n=5):
    """
    使用Semantic Scholar API查找领域专家
    """
    
    # Step 1: 搜索该领域的高被引论文
    papers = query_semantic_scholar(
        query=domain,
        sort="citation_count:desc",
        limit=50
    )
    
    # Step 2: 提取这些论文的作者
    authors_dict = {}
    for paper in papers:
        for author in paper["authors"]:
            if author["name"] not in authors_dict:
                authors_dict[author["name"]] = {
                    "name": author["name"],
                    "papers": [],
                    "total_citations": 0
                }
            authors_dict[author["name"]]["papers"].append(paper)
            authors_dict[author["name"]]["total_citations"] += paper["citation_count"]
    
    # Step 3: 排序并返回top N
    top_authors = sorted(
        authors_dict.values(),
        key=lambda x: x["total_citations"],
        reverse=True
    )[:top_n]
    
    return top_authors
```

**Token消耗**：0 token

---

### 2.3 查找领域大综述

```python
def find_domain_surveys(domain, keywords, top_n=3):
    """
    查找该领域的大综述
    """
    
    # Step 1: 构建综述查询
    survey_queries = [
        f"{domain} survey",
        f"{domain} review",
        f"{domain} overview",
        f"{keywords} systematic review"
    ]
    
    surveys = []
    
    for query in survey_queries:
        # 使用OpenAlex查找综述
        results = query_openalex(
            query=query,
            filters={
                "publication_year": ">2020",
                "type": ["review", "survey"]
            },
            sort="cited_by_count:desc",
            limit=top_n
        )
        
        surveys.extend(results)
    
    # Step 2: 去重和排序
    surveys = deduplicate_and_sort(surveys, by="cited_by_count")
    
    return surveys[:top_n]

# 示例
surveys = find_domain_surveys(
    domain="AI in Mental Health",
    keywords="LLM emotion recognition",
    top_n=3
)

# 输出：
# [
#   {
#     "title": "A Comprehensive Survey of AI Applications in Mental Health",
#     "authors": ["Johnson, M.", "Williams, K."],
#     "year": 2023,
#     "cited_by": 500,
#     "journal": "ACM Computing Surveys",
#     "abstract": "..."
#   },
#   ...
# ]
```

**Token消耗**：0 token

---

### 2.4 缓存背景知识

```python
def cache_background_knowledge(domain, experts, surveys):
    """
    缓存背景知识，避免重复查询
    """
    
    cache_file = Path("~/Desktop/playmore/background_knowledge_cache.json").expanduser()
    
    cache = {
        "domain": domain,
        "cached_at": datetime.now().isoformat(),
        "experts": experts,
        "surveys": surveys,
        "ttl": 30  # 30天过期
    }
    
    with open(cache_file, 'w') as f:
        json.dump(cache, f, indent=2)
    
    return cache

def get_cached_background_knowledge(domain):
    """
    获取缓存的背景知识
    """
    
    cache_file = Path("~/Desktop/playmore/background_knowledge_cache.json").expanduser()
    
    if not cache_file.exists():
        return None
    
    with open(cache_file, 'r') as f:
        cache = json.load(f)
    
    # 检查是否过期
    cached_at = datetime.fromisoformat(cache["cached_at"])
    ttl_days = cache.get("ttl", 30)
    
    if datetime.now() - cached_at > timedelta(days=ttl_days):
        return None  # 已过期
    
    if cache["domain"] == domain:
        return cache
    
    return None
```

---

### 2.5 生成背景知识摘要

```python
def generate_background_knowledge_summary(experts, surveys):
    """
    生成背景知识摘要，用于Prompt
    """
    
    summary = {
        "key_researchers": [],
        "key_papers": [],
        "key_surveys": [],
        "research_directions": [],
        "state_of_art": ""
    }
    
    # Step 1: 提取关键研究者
    for expert in experts[:3]:  # 只取前3个
        summary["key_researchers"].append({
            "name": expert["author"],
            "affiliation": expert["affiliation"],
            "citations": expert["cited_by_count"]
        })
    
    # Step 2: 提取关键论文
    for expert in experts:
        for paper in expert["papers"][:2]:  # 每个作者取2篇
            summary["key_papers"].append({
                "title": paper["title"],
                "authors": paper.get("authors", []),
                "year": paper["year"],
                "citations": paper["cited_by"],
                "journal": paper.get("journal")
            })
    
    # Step 3: 提取关键综述
    for survey in surveys[:2]:  # 只取前2个
        summary["key_surveys"].append({
            "title": survey["title"],
            "authors": survey["authors"],
            "year": survey["year"],
            "citations": survey["cited_by"]
        })
    
    # Step 4: 生成研究方向
    summary["research_directions"] = extract_research_directions(experts, surveys)
    
    # Step 5: 生成技术水平描述
    summary["state_of_art"] = generate_state_of_art_description(experts, surveys)
    
    return summary

def generate_state_of_art_description(experts, surveys):
    """
    生成该领域的技术水平描述
    """
    
    # 从关键论文中提取性能指标
    metrics = []
    for expert in experts:
        for paper in expert["papers"]:
            # 从论文标题或摘要中提取性能指标
            extracted_metrics = extract_metrics_from_paper(paper)
            metrics.extend(extracted_metrics)
    
    # 生成描述
    description = f"""
    该领域的当前技术水平：
    - 主要方法：{', '.join(extract_methods(experts))}
    - 性能范围：{get_performance_range(metrics)}
    - 主要挑战：{extract_challenges(surveys)}
    - 发展趋势：{extract_trends(experts, surveys)}
    """
    
    return description.strip()
```

---

## 第三部分：PDF提取方案

### 3.1 开源PDF提取项目对比

#### 项目1：PyPDF2（轻量级）

```python
import PyPDF2

def extract_pdf_with_pypdf2(pdf_path):
    """
    使用PyPDF2提取PDF
    """
    
    with open(pdf_path, 'rb') as file:
        reader = PyPDF2.PdfReader(file)
        
        text = ""
        for page in reader.pages:
            text += page.extract_text()
    
    return text

# 优点：
# - 轻量级，依赖少
# - 速度快
# - 易于安装

# 缺点：
# - 提取质量一般
# - 不支持复杂的PDF格式
# - 可能丢失格式信息
```

---

#### 项目2：pdfplumber（中等）

```python
import pdfplumber

def extract_pdf_with_pdfplumber(pdf_path):
    """
    使用pdfplumber提取PDF
    """
    
    with pdfplumber.open(pdf_path) as pdf:
        # 提取文本
        text = ""
        for page in pdf.pages:
            text += page.extract_text()
        
        # 提取表格
        tables = []
        for page in pdf.pages:
            page_tables = page.extract_tables()
            if page_tables:
                tables.extend(page_tables)
    
    return {
        "text": text,
        "tables": tables
    }

# 优点：
# - 提取质量好
# - 支持表格提取
# - 保留格式信息

# 缺点：
# - 比PyPDF2慢
# - 依赖较多
```

---

#### 项目3：pymupdf（高质量）

```python
import fitz  # pymupdf

def extract_pdf_with_pymupdf(pdf_path):
    """
    使用PyMuPDF提取PDF
    """
    
    doc = fitz.open(pdf_path)
    
    extracted = {
        "text": "",
        "metadata": doc.metadata,
        "sections": []
    }
    
    for page_num, page in enumerate(doc):
        # 提取文本
        text = page.get_text()
        extracted["text"] += text
        
        # 提取结构
        blocks = page.get_blocks()
        for block in blocks:
            if block[1]:  # 如果有文本
                extracted["sections"].append({
                    "page": page_num,
                    "text": block[4],
                    "type": "text" if block[0] == 0 else "image"
                })
    
    return extracted

# 优点：
# - 提取质量最好
# - 支持多种格式
# - 速度快
# - 支持元数据提取

# 缺点：
# - 依赖C库
# - 安装可能有问题
```

---

#### 项目4：Marker（AI驱动，省Token）

```python
# 使用Marker项目
# https://github.com/VikParuchuri/marker

from marker.convert import convert_single_pdf

def extract_pdf_with_marker(pdf_path):
    """
    使用Marker提取PDF（AI驱动，保留格式）
    """
    
    # Marker会自动识别PDF的结构
    # 输出Markdown格式，保留原始格式
    markdown_text = convert_single_pdf(pdf_path)
    
    return markdown_text

# 优点：
# - 使用AI识别PDF结构
# - 输出Markdown格式，易于处理
# - 保留原始格式和结构
# - 对学术论文优化

# 缺点：
# - 需要GPU加速（可选）
# - 首次运行较慢
# - 依赖较多

# 推荐：这个最适合学术论文提取！
```

---

### 3.2 推荐方案：Marker + 智能提取

```python
def extract_paper_sections_smart(pdf_path):
    """
    使用Marker提取PDF，然后智能提取关键部分
    """
    
    # Step 1: 使用Marker提取PDF
    markdown_text = convert_single_pdf(pdf_path)
    
    # Step 2: 解析Markdown，提取关键部分
    sections = {
        "title": extract_title(markdown_text),
        "abstract": extract_section(markdown_text, "abstract"),
        "introduction": extract_section(markdown_text, "introduction"),
        "method": extract_section(markdown_text, "method|methodology"),
        "experiments": extract_section(markdown_text, "experiment|evaluation"),
        "results": extract_section(markdown_text, "result"),
        "conclusion": extract_section(markdown_text, "conclusion"),
        "references": extract_section(markdown_text, "reference|bibliography")
    }
    
    # Step 3: 提取关键信息
    extracted = {
        "title": sections["title"],
        "abstract": sections["abstract"],
        "method_summary": summarize_section(sections["method"], max_tokens=200),
        "key_results": extract_key_results(sections["results"]),
        "key_figures": extract_figure_captions(markdown_text),
        "key_tables": extract_table_data(markdown_text)
    }
    
    return extracted

# Token消耗：
# - Marker提取：0 token（本地运行）
# - 关键信息提取：0 token（正则表达式）
# - 总计：0 token！
```

---

## 第四部分：改进的AI锐评流程

### 4.1 完整的锐评流程

```python
def generate_enhanced_review(paper, background_knowledge):
    """
    使用背景知识生成增强的AI锐评
    """
    
    # Step 1: 获取论文信息
    paper_info = {
        "title": paper["title"],
        "abstract": paper["abstract"],
        "keywords": paper["keywords"],
        "authors": paper["authors"],
        "year": paper["year"],
        "journal": paper["journal"],
        "cited_by": paper.get("cited_by", 0)
    }
    
    # Step 2: 如果有PDF，提取关键部分
    if paper.get("pdf_url"):
        pdf_sections = extract_paper_sections_smart(paper["pdf_url"])
        paper_info.update(pdf_sections)
    
    # Step 3: 构建增强的Prompt
    prompt = build_enhanced_review_prompt(
        paper_info=paper_info,
        background_knowledge=background_knowledge
    )
    
    # Step 4: 调用Claude生成锐评
    review = call_claude(prompt)
    
    # Step 5: 解析和结构化输出
    structured_review = parse_review_output(review)
    
    return structured_review

def build_enhanced_review_prompt(paper_info, background_knowledge):
    """
    构建增强的Prompt
    """
    
    prompt = f"""
你是一个在{background_knowledge['domain']}领域有10年经验的资深研究员。

该领域的关键研究者：
{format_researchers(background_knowledge['key_researchers'])}

该领域的关键论文：
{format_papers(background_knowledge['key_papers'])}

该领域的大综述：
{format_surveys(background_knowledge['key_surveys'])}

该领域的技术水平：
{background_knowledge['state_of_art']}

现在请评论以下论文：

标题：{paper_info['title']}
作者：{', '.join(paper_info['authors'])}
年份：{paper_info['year']}
期刊：{paper_info['journal']}
被引用次数：{paper_info['cited_by']}

摘要：
{paper_info['abstract']}

关键词：{', '.join(paper_info['keywords'])}

{f"方法总结：{paper_info.get('method_summary', '')}" if paper_info.get('method_summary') else ""}

{f"关键结果：{paper_info.get('key_results', '')}" if paper_info.get('key_results') else ""}

请从以下维度进行评论：

1. 核心贡献（2-3句）
   - 这篇论文的主要创新点是什么？
   - 相比{background_knowledge['key_researchers'][0]['name']}等人的工作有什么改进？
   - 改进的幅度有多大？

2. 技术评估（2-3句）
   - 使用的方法是否合理？
   - 实验设计是否严谨？
   - 结果是否支持结论？

3. 与相关工作的对比（2-3句）
   - 与关键论文相比如何？
   - 是否有重复的工作？
   - 创新点在哪里？

4. 应用前景（1-2句）
   - 这项工作有什么实际应用价值？
   - 可能的应用场景是什么？

5. 局限性和改进方向（2-3句）
   - 这项工作的主要局限是什么？
   - 后续研究可以如何改进？

6. 总体评价（1句）
   - 这篇论文是否值得阅读？

输出格式（JSON）：
{{
  "classification": "🔥必读 / 📖可读 / 💡了解",
  "core_contribution": "...",
  "technical_assessment": "...",
  "comparison": "...",
  "application": "...",
  "limitations": "...",
  "overall": "...",
  "recommendation": "必读 / 推荐 / 可选"
}}
"""
    
    return prompt
```

---

## 第五部分：集成到现有系统

### 5.1 修改playmore-review流程

```python
def playmore_review_enhanced(papers, keywords, seed_concepts):
    """
    增强的playmore-review
    """
    
    # Step 1: 获取或构建背景知识
    background_knowledge = get_or_build_background_knowledge(
        keywords=keywords,
        seed_concepts=seed_concepts
    )
    
    # Step 2: 对每篇论文生成增强的锐评
    reviews = []
    for paper in papers:
        review = generate_enhanced_review(paper, background_knowledge)
        reviews.append(review)
    
    # Step 3: 保存背景知识和锐评
    save_background_knowledge(background_knowledge)
    save_reviews(reviews)
    
    return reviews

def get_or_build_background_knowledge(keywords, seed_concepts):
    """
    获取缓存的背景知识，或构建新的
    """
    
    # 识别领域
    domain = identify_domain(keywords, seed_concepts)
    
    # 尝试获取缓存
    cached = get_cached_background_knowledge(domain)
    if cached:
        print(f"✓ 使用缓存的背景知识（{domain}）")
        return cached
    
    # 构建新的背景知识
    print(f"🔍 构建背景知识（{domain}）...")
    
    # 查找领域大牛和高分区文章
    print("  ├─ 查找领域大牛...")
    experts = find_domain_experts_and_papers(domain, top_n=3)
    
    # 查找领域大综述
    print("  ├─ 查找领域大综述...")
    surveys = find_domain_surveys(domain, keywords, top_n=3)
    
    # 生成背景知识摘要
    print("  └─ 生成背景知识摘要...")
    summary = generate_background_knowledge_summary(experts, surveys)
    
    # 缓存
    background_knowledge = {
        "domain": domain,
        "experts": experts,
        "surveys": surveys,
        "summary": summary
    }
    cache_background_knowledge(domain, experts, surveys)
    
    print(f"✓ 背景知识构建完成")
    
    return background_knowledge
```

---

## 第六部分：Token成本分析

### 6.1 成本对比

```
当前方案（无背景知识）：
- 每篇论文：500-700 token
- 10篇论文：5000-7000 token

改进方案1（优化Prompt + 结构化输出）：
- 每篇论文：800-1200 token
- 10篇论文：8000-12000 token
- 增加：3000-5000 token

改进方案2（+ 背景知识）：
- 背景知识构建：0 token（API调用）
- 每篇论文：1000-1500 token（包含背景知识）
- 10篇论文：10000-15000 token
- 增加：5000-8000 token

改进方案3（+ PDF提取）：
- PDF提取：0 token（Marker本地运行）
- 每篇论文：1200-1800 token（包含PDF信息）
- 10篇论文：12000-18000 token
- 增加：7000-11000 token

缓存效果：
- 第1次搜索：完整成本
- 第2-10次搜索（相同领域）：节省背景知识构建成本
- 长期平均节省：20-30%
```

---

## 第七部分：实现优先级

### Phase 1（第1-2周）：背景知识系统

```
任务：
1. 实现背景知识构建系统
   - 识别领域和概念
   - 查找领域大牛和高分区文章
   - 查找领域大综述
   - 缓存背景知识

2. 优化Prompt
   - 添加背景知识
   - 要求具体分析
   - 结构化输出

3. 集成到playmore-review

成本：0 token（API调用）
效果提升：50-100%
```

### Phase 2（第3周）：PDF提取

```
任务：
1. 集成Marker库
2. 实现智能部分提取
3. 集成到playmore-review

成本：0 token（本地运行）
效果提升：30-50%
```

### Phase 3（第4周）：优化和监控

```
任务：
1. 性能监控
2. 缓存优化
3. 用户反馈收集
```

---

## 第八部分：你的想法

### 关键问题

**1️⃣ 背景知识的来源**
- 使用OpenAlex API？
- 使用Semantic Scholar API？
- 两者都用？

**2️⃣ PDF提取工具**
- 使用Marker（推荐）？
- 使用PyMuPDF？
- 使用pdfplumber？

**3️⃣ 缓存策略**
- 按领域缓存？
- 缓存多久（30天）？
- 如何更新？

**4️⃣ 背景知识的深度**
- 查找多少个领域大牛（3-5个）？
- 每个大牛查找多少篇论文（2-3篇）？
- 查找多少个综述（2-3个）？

**5️⃣ 实现顺序**
- 先做背景知识系统？
- 再做PDF提取？
- 还是并行做？

