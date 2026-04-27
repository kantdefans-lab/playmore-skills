# 产品功能与用户交互设计 - 详细分析

## 第一部分：用户旅程分析

### 用户角色定义

#### 角色1：研究员（Primary User）

**背景**：
- 在某个领域进行深度研究
- 需要持续追踪最新论文
- 关心论文质量和相关性
- 有固定的研究方向

**需求**：
- 快速找到相关论文
- 自动分类和组织
- 长期追踪和对比
- 与团队协作

**使用频率**：每周5-10次搜索

**痛点**：
- 信息过载
- 手动分类耗时
- 容易遗漏重要论文
- 难以追踪概念演进

---

#### 角色2：学生（Secondary User）

**背景**：
- 写论文或做课程项目
- 需要快速了解领域
- 时间有限
- 预算有限

**需求**：
- 快速搜索和总结
- 简单易用的界面
- 低成本
- 快速生成笔记

**使用频率**：每周2-3次搜索

**痛点**：
- 不知道从哪里开始
- 论文太多，不知道读哪个
- 手动记笔记太慢
- 成本太高

---

#### 角色3：产品经理（Tertiary User）

**背景**：
- 追踪行业趋势
- 了解竞争对手
- 发现新技术
- 做决策参考

**需求**：
- 快速获取行业动态
- 趋势分析
- 竞争对手追踪
- 数据可视化

**使用频率**：每周1-2次搜索

**痛点**：
- 信息来源分散
- 难以识别趋势
- 难以对比不同方向

---

### 用户旅程地图

#### 研究员的完整旅程

```
第1天：初始化
  ├─ 定义研究方向
  │  └─ /playmore init
  │     输入：种子概念（"AI×心理健康"）
  │     输出：概念库初始化
  │
  └─ 首次搜索
     └─ /playmore AI 心理健康
        输入：关键词
        输出：论文列表 + 概念库

第2-7天：日常使用
  ├─ 定期搜索
  │  └─ /playmore AI 心理健康
  │     输入：相同或相似关键词
  │     输出：新论文 + 更新的概念库
  │
  ├─ 审核新概念
  │  └─ /playmore-concepts-review
  │     输入：待审核概念列表
  │     输出：批准/拒绝/编辑
  │
  ├─ 浏览论文笔记
  │  └─ 打开 ~/Desktop/playmore/AI×心理健康/
  │     查看论文笔记和概念链接
  │
  └─ 查看概念库
     └─ /playmore-concepts list
        查看所有概念和关系

第8天：周总结
  ├─ 查看周报
  │  └─ /playmore-weekly-report
  │     输出：本周新论文、新概念、趋势
  │
  └─ 导出笔记
     └─ /playmore-export --format markdown
        导出到Obsidian或其他工具
```

---

## 第二部分：核心功能模块

### 模块1：搜索和发现

#### 功能1.1：多源搜索

```
用户输入：/playmore AI 心理健康

系统动作：
1. 并行调用多个API
   ├─ OpenAlex
   ├─ Semantic Scholar
   ├─ Google Scholar
   └─ CNKI

2. 去重和合并
   ├─ 按DOI去重
   ├─ 按标题相似度去重
   └─ 保留最完整的元数据

3. 打分和排序
   ├─ 关键词匹配
   ├─ 期刊分区
   └─ 引用数量

4. 返回结果
   ├─ 前30篇论文
   ├─ 每篇的详细信息
   └─ 推荐阅读顺序
```

**用户体验**：
```
$ /playmore AI 心理健康

🔍 搜索中...
  ├─ OpenAlex: 45篇
  ├─ Semantic Scholar: 38篇
  ├─ Google Scholar: 52篇
  └─ CNKI: 12篇

✓ 找到 147 篇论文

📊 去重和排序中...
✓ 保留 30 篇高质量论文

📋 结果：
  1. 🔥 LLM-based Therapy Assistant (2024)
     作者：Smith, J. | 期刊：Nature Mental Health
     引用：15 | 相关度：0.95
     
  2. 🔥 Multimodal Learning for Mental Health (2024)
     作者：Johnson, M. | 期刊：IEEE TMI
     引用：8 | 相关度：0.92
     
  ...
```

---

#### 功能1.2：定时搜索

```
用户设置：
/playmore-schedule add "AI therapy" from 2026-05-01 to 2026-05-31

系统动作：
1. 每天自动搜索
2. 检查是否有新论文
3. 如果有新论文，发送通知
4. 自动添加到概念库

用户体验：
$ /playmore-schedule list

📅 定时任务：
  1. AI therapy
     ├─ 日期范围：2026-05-01 ~ 2026-05-31
     ├─ 状态：运行中
     ├─ 最后运行：2026-04-27 10:30
     └─ 新论文：3篇

  2. Emotion recognition
     ├─ 日期范围：2026-04-01 ~ 2026-06-30
     ├─ 状态：运行中
     ├─ 最后运行：2026-04-27 09:00
     └─ 新论文：0篇
```

---

### 模块2：概念库管理

#### 功能2.1：自动概念识别

```
用户搜索后，系统自动识别概念

用户体验：
$ /playmore AI 心理健康

✓ 找到 30 篇论文

🏷️  识别概念中...
  ├─ 匹配现有概念：2个
  │  ├─ AI×心理健康 (relevance: 0.95)
  │  └─ Transformer模型 (relevance: 0.78)
  │
  └─ 识别新概念：3个
     ├─ Prompt Engineering (confidence: 0.85) ⏳ 待审核
     ├─ Multimodal Learning (confidence: 0.72) ⏳ 待审核
     └─ Physiological Signal Processing (confidence: 0.90) ✓ 自动批准

💡 提示：有 2 个新概念待审核
   运行 /playmore-concepts-review 查看
```

---

#### 功能2.2：概念审核界面

```
用户运行：/playmore-concepts-review

系统显示：
┌─────────────────────────────────────────────────────┐
│ 待审核概念（2个）                                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│ 1️⃣  Prompt Engineering                             │
│    ├─ 置信度：0.85                                  │
│    ├─ 发现于：3篇论文                               │
│    ├─ 描述：设计有效的提示词来指导AI模型            │
│    ├─ 关键词：prompt, instruction, engineering     │
│    ├─ 上级概念：AI×心理健康                         │
│    │                                               │
│    └─ 操作：                                        │
│       [A] 批准  [R] 拒绝  [E] 编辑  [S] 跳过       │
│                                                     │
│ 2️⃣  Multimodal Learning                            │
│    ├─ 置信度：0.72                                  │
│    ├─ 发现于：2篇论文                               │
│    ├─ 描述：融合多种模态的学习方法                  │
│    ├─ 关键词：multimodal, fusion, cross-modal      │
│    ├─ 上级概念：AI×心理健康                         │
│    │                                               │
│    └─ 操作：                                        │
│       [A] 批准  [R] 拒绝  [E] 编辑  [S] 跳过       │
│                                                     │
└─────────────────────────────────────────────────────┘

用户输入：A
✓ 已批准 Prompt Engineering

用户输入：E
编辑模式：
  名称：Multimodal Learning
  描述：[编辑中...]
  关键词：[编辑中...]
  
✓ 已保存，自动批准
```

---

#### 功能2.3：概念浏览和搜索

```
用户运行：/playmore-concepts list

系统显示：
📚 概念库（48个概念）

🌱 种子概念（2个）
  ├─ AI×心理健康 (12篇论文)
  └─ 深度学习×情绪识别 (8篇论文)

🤖 自动概念（46个）
  ├─ 🔥 热门概念（命中率>5）
  │  ├─ Transformer模型 (9篇)
  │  ├─ Prompt Engineering (7篇)
  │  ├─ Multimodal Learning (5篇)
  │  └─ ...
  │
  ├─ 📖 常见概念（命中率2-5）
  │  ├─ Physiological Signal Processing (4篇)
  │  ├─ Depression Detection (3篇)
  │  └─ ...
  │
  └─ 💡 新概念（命中率1）
     ├─ Graph Neural Networks (1篇)
     ├─ Reinforcement Learning (1篇)
     └─ ...

用户运行：/playmore-concepts search "transformer"

搜索结果：
  ✓ Transformer模型
    ├─ 类型：自动概念
    ├─ 置信度：0.92
    ├─ 论文数：9
    ├─ 相关概念：Attention Mechanism, BERT, GPT
    └─ 最后更新：2026-04-27
```

---

### 模块3：论文笔记和知识管理

#### 功能3.1：自动生成笔记

```
用户搜索后，系统自动生成结构化笔记

生成的笔记位置：
~/Desktop/playmore/AI×心理健康/2024_smith_llm-therapy.md

笔记内容：
---
title: LLM-based Therapy Assistant
authors: [Smith, J., Johnson, M.]
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

## 相关论文
- [[2023_zhang_emotion-detection]]
- [[2024_williams_multimodal-learning]]
```

---

#### 功能3.2：笔记浏览和导航

```
用户打开笔记目录：~/Desktop/playmore/

📁 playmore/
├─ 📄 INDEX.md (总索引)
├─ 📄 WEEKLY_REPORT.md (周报)
│
├─ 📁 AI×心理健康/ (12篇论文)
│  ├─ 📄 INDEX.md (概念索引)
│  ├─ 📄 2024_smith_llm-therapy.md
│  ├─ 📄 2024_johnson_multimodal.md
│  └─ ...
│
├─ 📁 深度学习×情绪识别/ (8篇论文)
│  ├─ 📄 INDEX.md
│  ├─ 📄 2023_zhang_emotion-detection.md
│  └─ ...
│
└─ 📁 concepts/ (概念库)
   ├─ 📄 INDEX.md (概念总索引)
   ├─ 📄 seed_001.md (AI×心理健康)
   ├─ 📄 auto_001.md (Transformer模型)
   └─ ...

用户在Obsidian中打开笔记，可以：
- 点击 [[概念名]] 跳转到概念笔记
- 点击 [[论文名]] 跳转到论文笔记
- 查看概念之间的关系图
```

---

### 模块4：数据可视化和分析

#### 功能4.1：概念关系图

```
用户运行：/playmore-concepts visualize

系统生成：
- 概念关系图（可视化）
- 显示概念之间的层级和关系
- 支持交互式探索

可视化示例：
                    AI×心理健康
                         |
                    _____|_____
                   |           |
            Transformer    Prompt
            模型          Engineering
                |              |
                |______________|
                       |
                Multimodal
                Learning
                       |
            Physiological Signal
            Processing
```

---

#### 功能4.2：周报和趋势分析

```
用户运行：/playmore-weekly-report

系统生成：
📊 周报（2026-04-21 ~ 2026-04-27）

📈 统计数据
  ├─ 新论文：12篇
  ├─ 新概念：3个
  ├─ 已批准概念：2个
  └─ 总概念数：48个

🔥 热门概念
  ├─ Transformer模型 (+2篇)
  ├─ Prompt Engineering (+1篇)
  └─ Multimodal Learning (+1篇)

📚 推荐阅读
  ├─ 🔥 LLM-based Therapy Assistant
  ├─ 🔥 Multimodal Learning for Mental Health
  └─ 📖 Emotion Recognition from Text

📉 冷门概念
  ├─ Graph Neural Networks (0篇)
  ├─ Reinforcement Learning (0篇)
  └─ 建议考虑归档

💡 洞察
  ├─ Transformer模型在心理健康领域应用增加
  ├─ 多模态学习成为新的研究方向
  └─ 生理信号处理开始受关注
```

---

## 第三部分：高级功能

### 功能5：协作和分享

#### 功能5.1：团队协作

```
用户邀请团队成员：
/playmore-team add john@example.com

团队成员可以：
- 查看共享的概念库
- 批准/拒绝概念
- 添加个人笔记
- 评论论文

用户体验：
$ /playmore-team list

👥 团队成员（3人）
  ├─ 你 (所有者)
  ├─ John (编辑)
  └─ Jane (查看者)

$ /playmore-team share concept "AI×心理健康"

✓ 已分享给 John 和 Jane
```

---

#### 功能5.2：论文讨论

```
用户在论文笔记中添加讨论：

# 讨论

## 评论 1
作者：John
时间：2026-04-27 10:30
内容：这篇论文的方法很有创意，但实验数据不足

## 评论 2
作者：你
时间：2026-04-27 11:00
内容：同意，但我认为这是初步研究，后续应该会有更多数据

## 评论 3
作者：Jane
时间：2026-04-27 11:30
内容：建议关注他们的后续工作
```

---

### 功能6：个性化推荐

#### 功能6.1：论文推荐

```
系统基于用户的研究方向推荐论文

用户体验：
$ /playmore-recommend

🎯 为你推荐（基于你的研究方向）

⭐ 高度相关（相关度 > 0.9）
  ├─ 2024_smith_llm-therapy.md (0.95)
  ├─ 2024_johnson_multimodal.md (0.92)
  └─ ...

📖 中等相关（相关度 0.7-0.9）
  ├─ 2023_zhang_emotion-detection.md (0.85)
  ├─ 2024_williams_physiological.md (0.78)
  └─ ...

💡 可能感兴趣（相关度 0.5-0.7）
  ├─ 2024_brown_graph-neural.md (0.65)
  └─ ...
```

---

#### 功能6.2：概念推荐

```
系统推荐相关的新概念

用户体验：
$ /playmore-concepts recommend

🎯 推荐新概念

基于你的研究方向，以下概念可能感兴趣：

1. Attention Mechanism
   ├─ 相关度：0.88
   ├─ 理由：与 Transformer模型 密切相关
   ├─ 论文数：5
   └─ 建议：添加到概念库

2. Fine-tuning
   ├─ 相关度：0.82
   ├─ 理由：与 Prompt Engineering 相关
   ├─ 论文数：3
   └─ 建议：添加到概念库
```

---

## 第四部分：用户交互流程设计

### 流程1：新用户入门

```
第1步：初始化
  用户：/playmore init
  系统：
    1. 询问研究方向
    2. 建议种子概念
    3. 创建概念库
    4. 生成配置文件

第2步：首次搜索
  用户：/playmore AI 心理健康
  系统：
    1. 搜索论文
    2. 识别概念
    3. 生成笔记
    4. 显示结果

第3步：审核概念
  用户：/playmore-concepts-review
  系统：
    1. 显示待审核概念
    2. 用户批准/拒绝
    3. 更新概念库

第4步：浏览笔记
  用户：打开 ~/Desktop/playmore/
  系统：
    1. 显示论文笔记
    2. 显示概念链接
    3. 支持Obsidian集成
```

---

### 流程2：日常使用

```
每日工作流：

上午：
  1. 检查定时搜索结果
     /playmore-schedule status
  
  2. 审核新概念
     /playmore-concepts-review
  
  3. 浏览新论文
     打开 ~/Desktop/playmore/

下午：
  1. 进行新搜索
     /playmore [新关键词]
  
  2. 阅读论文笔记
     在Obsidian中浏览
  
  3. 添加个人笔记
     编辑论文笔记

周末：
  1. 查看周报
     /playmore-weekly-report
  
  2. 分析趋势
     /playmore-concepts visualize
  
  3. 导出笔记
     /playmore-export --format markdown
```

---

## 第五部分：功能优先级

### MVP（最小可行产品）

```
Phase 1（第1-2周）：核心功能
  ✅ 多源搜索（fetch）
  ✅ 打分排序（score）
  ✅ AI点评（review）
  ✅ 自动笔记生成（notes）
  ✅ 基础缓存系统

Phase 2（第3-4周）：概念库
  ✅ 自动概念识别（concepts-extract）
  ✅ 概念审核界面（concepts-review）
  ✅ 概念浏览（concepts list）
  ✅ 概念链接（在笔记中）

Phase 3（第5-6周）：优化和监控
  ✅ 分层模式（fast/standard/deep）
  ✅ 性能监控
  ✅ 缓存优化
  ✅ 用户文档
```

---

### 后续功能（V2）

```
Phase 4（第7-8周）：高级功能
  ⏳ 定时搜索（schedule）
  ⏳ 周报和趋势分析
  ⏳ 概念关系可视化
  ⏳ 论文推荐

Phase 5（第9-10周）：协作功能
  ⏳ 团队协作
  ⏳ 论文讨论
  ⏳ 权限管理
  ⏳ 活动日志

Phase 6（第11-12周）：个性化
  ⏳ 个性化推荐
  ⏳ 用户偏好设置
  ⏳ 高级搜索过滤
  ⏳ 自定义工作流
```

---

## 第六部分：用户反馈和迭代

### 反馈收集机制

```
1. 内置反馈
   /playmore-feedback "这个功能很好用"
   
2. 使用统计
   - 记录每个功能的使用频率
   - 记录用户的操作路径
   - 识别常见的错误操作

3. 用户调查
   - 定期发送问卷
   - 收集满意度评分
   - 了解用户需求

4. 社区讨论
   - GitHub Issues
   - 讨论区
   - 用户反馈表单
```

---

### 迭代循环

```
第1周：收集反馈
  - 用户使用统计
  - 错误日志分析
  - 用户反馈汇总

第2周：分析和规划
  - 识别主要问题
  - 优先级排序
  - 制定改进计划

第3周：实现改进
  - 修复bug
  - 优化性能
  - 添加新功能

第4周：发布和验证
  - 发布新版本
  - 收集用户反馈
  - 验证改进效果
```

---

## 总结：完整的产品体验

### 用户的完整旅程

```
Day 1: 初始化
  /playmore init
  → 创建概念库

Day 1-7: 日常使用
  /playmore [关键词]
  → 搜索论文
  → 识别概念
  → 生成笔记
  → 审核概念

Day 8: 周总结
  /playmore-weekly-report
  → 查看趋势
  → 发现新方向

Week 2+: 持续优化
  /playmore-concepts visualize
  → 查看概念关系
  → 发现研究机会
  → 与团队协作
```

### 核心价值主张

```
对于研究员：
  ✅ 节省50%的文献综述时间
  ✅ 自动组织和分类论文
  ✅ 发现隐藏的研究方向
  ✅ 与团队协作

对于学生：
  ✅ 快速了解领域
  ✅ 自动生成笔记
  ✅ 低成本（缓存优化）
  ✅ 易于使用

对于产品经理：
  ✅ 追踪行业趋势
  ✅ 发现竞争对手
  ✅ 数据驱动决策
  ✅ 可视化分析
```

