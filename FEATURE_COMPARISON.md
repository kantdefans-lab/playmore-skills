# dailypaper-skills vs playmore-skills 功能对比

## 核心架构差异

### dailypaper-skills（原始项目）
- **设计理念**：生产级别的完整系统，针对机器人/AI研究领域
- **数据流**：多步骤流水线，每步通过临时JSON文件传递数据
- **代码量**：~4300行Python代码 + SKILL.md定义
- **依赖**：Zotero、Obsidian、arXiv、HuggingFace、Chrome DevTools

### playmore-skills（当前项目）
- **设计理念**：轻量级skill套件，通用学术论文追踪
- **数据流**：skill间直接调用，内存传递
- **代码量**：仅SKILL.md定义，无实现代码
- **依赖**：gs-skills、cnki-skills、OpenAlex、Semantic Scholar

---

## 功能详细对比

### 1️⃣ 数据源与搜索

| 功能 | dailypaper-skills | playmore-skills | 作用 |
|------|------------------|-----------------|------|
| **HuggingFace Daily Papers API** | ✅ 有 | ❌ 无 | 自动获取HF热门论文，无需手动搜索 |
| **HuggingFace Trending API** | ✅ 有 | ❌ 无 | 获取trending论文，带upvotes热度数据 |
| **arXiv API** | ✅ 有 | ❌ 无 | 直接查询arXiv，支持分类过滤 |
| **OpenAlex API** | ❌ 无 | ✅ 有 | 通用学术数据库，覆盖面广 |
| **Semantic Scholar API** | ❌ 无 | ✅ 有 | 学术论文搜索，有引用计数 |
| **Google Scholar** | ❌ 无 | ✅ 有 | 通用学术搜索，需Chrome DevTools |
| **CNKI（知网）** | ❌ 无 | ✅ 有 | 中文论文搜索，需Chrome DevTools |

**结论**：
- dailypaper-skills 专注于**自动化推荐**（HF Daily + arXiv），适合被动接收
- playmore-skills 专注于**主动搜索**（多源API），适合主动查询

---

### 2️⃣ 打分与排序

#### dailypaper-skills 的打分逻辑（fetch_and_score.py - 472行）

```python
# 关键词匹配
- 标题命中：+3 pts
- 摘要命中：+1 pt
- 负面关键词：直接排除（-999）

# 领域加权
- domain_boost_keywords 命中：+1~2 pts

# Trending加权（仅HF Trending）
- upvotes 5-10：+1 pt
- upvotes 10-20：+2 pts
- upvotes 20+：+3 pts
- 相关论文额外加分

# 跨天去重
- 单天模式：跟 .history.json 交叉去重
- 周末模式：保留5+ upvotes热门论文作为"再推荐"
- 多天模式：跳过历史去重，回补候选

# 最终筛选
- 低于 MIN_SCORE 排除
- 保留 TOP_N（默认30）
```

#### playmore-skills 的打分逻辑（SKILL.md）

```python
# 关键词匹配
- 标题命中：+2 pts（最多4）
- 摘要命中：+1 pt（最多3）
- 负面关键词：排除

# 期刊分区
- SCI Q1：4 pts
- SCI Q2：3 pts
- SCI Q3：2 pts
- CSSCI/北大核心：1 pt

# Q3+加权
- 期刊≥Q3 且关键词>0：+2 pts

# 最终筛选
- 低于2分排除
- 保留TOP 30
```

**差异分析**：
| 维度 | dailypaper-skills | playmore-skills |
|------|------------------|-----------------|
| 关键词权重 | 标题+3，摘要+1 | 标题+2，摘要+1 |
| 领域加权 | ✅ 有（domain_boost） | ❌ 无 |
| Trending热度 | ✅ 有（upvotes） | ❌ 无 |
| 期刊分区 | ❌ 无 | ✅ 有 |
| 跨天去重 | ✅ 有（.history.json） | ❌ 无 |
| 历史回补 | ✅ 有 | ❌ 无 |

**结论**：
- dailypaper-skills：**时间序列感知**，适合日推荐（避免重复推荐同一篇）
- playmore-skills：**质量导向**，适合一次性搜索（强调期刊等级）

---

### 3️⃣ 元数据富化

#### dailypaper-skills 的富化（enrich_papers.py - 569行）

**并发HTTP请求**（asyncio + curl，Semaphore=10）：
- 从arXiv HTML提取首图URL
- 提取作者、机构、章节标题、图表标题
- 提取方法名（带stop words过滤）
- 检测是否有真机实验（real-world keywords）
- 机构提取失败时fallback到pdftotext

**输出字段**：
```json
{
  "image_url": "...",
  "authors": [...],
  "affiliations": [...],
  "method_names": ["VLA", "Diffusion Policy"],
  "has_real_world": true,
  "sections": ["Introduction", "Method", ...],
  "figures": ["Figure 1: ...", ...]
}
```

#### playmore-skills 的富化

- ❌ 无专门的富化步骤
- 依赖各API返回的元数据

**结论**：
- dailypaper-skills 做了**深度HTML解析**，提取结构化信息
- playmore-skills 依赖API返回的数据

---

### 4️⃣ 图片管理

#### dailypaper-skills（download_note_images.py - 284行）

```python
# 多路获取策略
1. arXiv HTML <figure> 标签（优先）
2. 项目主页 teaser 图
3. PDF 提取（pdfimages -png）
4. 写完后检查可达性，不可达自动下载到本地

# 输出
- 本地存储：~/ObsidianVault/论文笔记/.images/
- Markdown 引用：![](file:///path/to/image.png)
```

#### playmore-skills

- ❌ 无图片管理

**结论**：
- dailypaper-skills 做了**完整的图片生命周期管理**
- playmore-skills 不处理图片

---

### 5️⃣ 概念库维护

#### dailypaper-skills（自动分类 + 索引）

**16个概念分类**：
```
_概念/
├── 1-生成模型/          (DiT, Flow Matching, ...)
├── 2-强化学习/          (PPO, DQN, ...)
├── 3-机器人策略/        (Diffusion Policy, ...)
├── 4-3D视觉/            (3D Gaussian, NeRF, ...)
├── 5-仿真器/            (Isaac Sim, MuJoCo, ...)
├── 6-数据集/            (RLDS, Bridge, ...)
├── ... (共16个)
└── 0-待分类/
```

**功能**：
- 自动从论文笔记中提取 `[[概念]]` 链接
- 自动分类到16个子目录
- 生成概念笔记（定义 + 数学形式 + 代表工作）
- 生成MOC索引页（moc_builder.py）

#### playmore-skills

- ❌ 无概念库维护

**结论**：
- dailypaper-skills 做了**知识图谱构建**
- playmore-skills 不维护概念库

---

### 6️⃣ 历史管理与去重

#### dailypaper-skills（.history.json）

```json
{
  "2026-04-26": [
    {"arxiv_id": "2404.12345", "title": "..."},
    ...
  ],
  "2026-04-25": [...]
}
```

**功能**：
- 跨天去重（避免同一篇论文重复推荐）
- 周末模式：保留5+ upvotes的热门论文作为"再推荐"
- 多天模式：自动回补候选（不足20篇时从历史补充）
- 自动清理（只保留最近30天）

#### playmore-skills

- ❌ 无历史管理

**结论**：
- dailypaper-skills 做了**时间序列管理**
- playmore-skills 每次搜索独立

---

### 7️⃣ 定时任务管理

#### dailypaper-skills（playmore-schedule 等价物）

- ❌ 在dailypaper-skills中没有看到

#### playmore-skills（playmore-schedule）

```bash
/playmore-schedule add "AI therapy" from 2026-05-01 to 2026-05-31
/playmore-schedule list
/playmore-schedule edit task_001
/playmore-schedule delete task_001
/playmore-schedule run task_001
```

**功能**：
- 定义日期范围和关键词
- 自动或手动触发
- 存储在 `schedule.json`

**结论**：
- playmore-skills 有**定时任务管理**
- dailypaper-skills 没有（或在其他地方）

---

### 8️⃣ 批量处理守护进程

#### dailypaper-skills（paper_daemon.py - 788行）

```bash
python3 paper_daemon.py -c "VLA"     # 处理VLA分类
python3 paper_daemon.py --status     # 查看进度
python3 paper_daemon.py --list       # 列出所有分类
```

**功能**：
- 从Zotero获取指定分类的论文列表（递归子分类）
- 逐篇调用Claude Code处理
- **Rate limit处理**：指数退避（60s → 最大12h）
- **配额监控**：每3篇检查一次，>85%自动等待
- **断点续传**：checkpoint持久化
- **进程锁**：防止并发
- **自动跳过**：已有笔记（>100行）

#### playmore-skills

- ❌ 无批量处理守护进程

**结论**：
- dailypaper-skills 做了**生产级别的后台处理**
- playmore-skills 没有

---

### 9️⃣ 配置管理

#### dailypaper-skills（user_config.py - 212行）

```python
# 集中配置
DEFAULT_CONFIG = {
    "paths": {
        "obsidian_vault": "~/ObsidianVault",
        "paper_notes_folder": "论文笔记",
        "daily_papers_folder": "DailyPapers",
        "concepts_folder": "_概念",
        "zotero_db": "~/Zotero/zotero.sqlite",
        "zotero_storage": "~/Zotero/storage",
    },
    "daily_papers": {
        "keywords": [...],
        "negative_keywords": [...],
        "domain_boost_keywords": [...],
        "arxiv_categories": ["cs.RO", "cs.CV", "cs.AI", "cs.LG"],
        "min_score": 2,
        "top_n": 30
    },
    "automation": {
        "auto_refresh_indexes": true,
        "git_commit": false,
        "git_push": false
    }
}
```

**功能**：
- 集中管理所有路径、关键词、打分规则
- 配置校验（如git_push不能在git_commit关闭时开启）
- LRU缓存加速

#### playmore-skills（config.example.json）

```json
{
  "keywords": [...],
  "negative_keywords": [...]
}
```

**功能**：
- 简单的关键词配置

**结论**：
- dailypaper-skills 做了**完整的配置管理系统**
- playmore-skills 只有基础配置

---

### 🔟 Zotero集成

#### dailypaper-skills（zotero_helper.py - 421行）

```python
# 功能
- 查询Zotero数据库（SQLite）
- 获取指定分类的论文列表（递归子分类）
- 定位PDF文件或在线源
- 批量处理分类

# 输出
- 论文元数据
- PDF路径或在线链接
```

#### playmore-skills

- ❌ 无Zotero集成

**结论**：
- dailypaper-skills 做了**Zotero深度集成**
- playmore-skills 不支持

---

## 功能总结表

| 功能模块 | dailypaper-skills | playmore-skills | 优先级 |
|---------|------------------|-----------------|--------|
| **数据源** | | | |
| HuggingFace Daily | ✅ | ❌ | 中 |
| arXiv API | ✅ | ❌ | 中 |
| OpenAlex API | ❌ | ✅ | 中 |
| Semantic Scholar | ❌ | ✅ | 中 |
| Google Scholar | ❌ | ✅ | 低 |
| CNKI | ❌ | ✅ | 低 |
| **打分系统** | | | |
| 关键词匹配 | ✅ | ✅ | 高 |
| 期刊分区 | ❌ | ✅ | 中 |
| 领域加权 | ✅ | ❌ | 中 |
| Trending热度 | ✅ | ❌ | 低 |
| 跨天去重 | ✅ | ❌ | 中 |
| **元数据** | | | |
| HTML富化 | ✅ | ❌ | 低 |
| 机构提取 | ✅ | ❌ | 低 |
| 方法名提取 | ✅ | ❌ | 低 |
| 真机实验检测 | ✅ | ❌ | 低 |
| **图片管理** | ✅ | ❌ | 低 |
| **概念库** | ✅ | ❌ | 中 |
| **历史管理** | ✅ | ❌ | 中 |
| **定时任务** | ❌ | ✅ | 中 |
| **守护进程** | ✅ | ❌ | 低 |
| **Zotero集成** | ✅ | ❌ | 低 |
| **配置管理** | ✅ | ❌ | 低 |

---

## 建议

### 🎯 如果你想要 playmore-skills 更强大

**高优先级**（建议添加）：
1. **跨天去重** - 避免重复推荐同一篇论文
2. **概念库维护** - 构建知识图谱
3. **历史管理** - 追踪已推荐的论文

**中优先级**（可选）：
1. **领域加权** - 针对特定领域的论文加权
2. **HTML富化** - 提取更多结构化信息
3. **图片管理** - 自动下载和管理论文图片

**低优先级**（可忽略）：
1. 守护进程 - 除非需要大规模批量处理
2. Zotero集成 - 除非你用Zotero管理论文库
3. Trending热度 - 仅对HF Daily有用

### 🎯 如果你想保持 playmore-skills 轻量级

- 保持现状，专注于**多源搜索 + 质量打分**
- 定时任务已经很好用了
- 用户可以手动管理历史和概念

