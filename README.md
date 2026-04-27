# playmore-skills

<p align="center">
  <a href="https://github.com/kantdefans-lab/playmore-skills/stargazers"><img src="https://img.shields.io/github/stars/kantdefans-lab/playmore-skills?style=social" alt="GitHub Stars"></a>
  <a href="https://github.com/kantdefans-lab/playmore-skills/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="License"></a>
  <img src="https://img.shields.io/badge/platform-Qclaw%20%7C%20Claude%20Code%20%7C%20Codex-brightgreen" alt="Platform">
  <img src="https://img.shields.io/badge/VPN-not%20required-success" alt="No VPN">
  <img src="https://img.shields.io/badge/sources-Google%20Scholar%20%7C%20CNKI%20%7C%20OpenAlex%20%7C%20Semantic%20Scholar-orange" alt="Sources">
</p>

<p align="center"><b>外面阳光真好，出去玩吧。</b></p>
<p align="center">首个面向社科研究生的 AI 论文检索分析系统</p>

<p align="center">
  <a href="#中文">中文</a> · <a href="#english">English</a>
</p>

---

## 最新动态

- **2026-04** 🎉 playmore-skills 正式发布，支持谷歌学术、知网、OpenAlex、Semantic Scholar 四源搜索
- **2026-04** 支持 Qclaw / Claude Code / Codex 三平台

## 路线图

- [x] 四源跨平台搜索与去重
- [x] 三维打分 + AI 点评
- [x] 概念分层笔记存储
- [x] 定时任务管理
- [ ] 引用链追踪：找到一篇好文章后，自动往上追它引用了谁、往下追谁引用了它
- [ ] 研究空白识别：分析一批文献后，AI 指出哪些角度还没人做过
- [ ] 多项目隔离：同时跟多个研究方向，笔记和任务互不干扰
- [ ] 笔记全文搜索：在所有已保存的笔记里搜关键词，快速找到之前读过的内容

---

<a id="中文"></a>

## 中文

这是首个专门为社科研究生、博士生做的 AI 论文检索分析系统。

你不需要 VPN，不需要懂编程，不需要注册任何海外账号。你只需要一部绑了微信的手机，就能让 AI 帮你同时搜四个数据库、自动打分筛选、写好结构化笔记，存到你桌面上。

查文献这件事，本来就不应该占你那么多时间。

---

### 三个原则

**第一，让所有社科同学都能以最简单的方式用上AI辅助。**

现在很多 AI 工具要翻墙，要信用卡，要折腾一堆配置。我们不走这条路。这个系统选的每一个工具、走的每一步操作，都是在"不翻墙、不写代码、能长期用"这个前提下反复打磨出来的。你的同学里有人完全不懂技术，这套流程他也能跑起来。

**第二，用最少的 token 实现最好的效果。**

AI 用起来是有成本的，每次对话都在消耗额度。我们不做那种靠大量prompt和ai屎山代码来硬挤出好效果的skill。我会认真思考playmore的每个部分最高效率的保证最低token。

**第三，让 AI 替你干活，把时间还给你自己。**

从搜索、去重、打分、点评到保存笔记，整个流程是自动的。你输入一个主题，剩下的事情交给它。你不需要一篇一篇打开谷歌学术，不需要手动记录哪篇看过哪篇没看，不需要在"我还没看完文献"的焦虑里耗着。

---

### 它能做什么

- 同时搜索谷歌学术、知网、OpenAlex、Semantic Scholar，按 DOI 和标题自动去重，不会给你重复的结果
- 三个维度自动打分：关键词相关度 + 期刊分区（SCI/CSSCI/北大核心）+ 高分区加权，低于门槛的直接过滤掉
- AI 自动给每篇论文分级：⭐ 必读 / 📖 可读 / 💡 了解，并写一段点评说明理由
- 按你的研究概念自动建文件夹，比如"AI×心理健康"、"数字经济×劳动力市场"，笔记分类存好
- 可以设定时任务，比如"每个月自动搜一次这个主题的新文献"，到时间自动跑
- 每篇笔记都记录 PDF 链接、DOI、原文页面，找原文不用再重新搜

---

### 开始使用

#### 先搞清楚这是什么

这个系统是一套"skill"，运行在 AI 命令行工具里。你可以把它理解成：给 AI 装了一个专门查论文的插件。你跟 AI 说"帮我查 AI 心理健康的文献"，它就知道该怎么做了。

"命令行工具"听起来很技术，但实际上就是一个你可以打字跟 AI 对话的窗口，和微信聊天差不多，只是多了一些特殊指令。

我们推荐用 **Qclaw**，原因下面说。

---

#### 第一步：下载 Qclaw

**为什么是 Qclaw，不是 Claude 或 ChatGPT？**

Claude 和 ChatGPT 都需要翻墙，而且要绑海外信用卡付费。Qclaw 是国内厂商做的，基于微信生态，用手机微信扫码就能登录，有免费额度，不需要 VPN，不需要信用卡。

Qclaw 的 AI 模型能力确实不如 Claude、GPT 这些国际厂商——但这没关系。playmore 做的事情是信息检索和结构化整理，不是写论文、不是做复杂推理，对模型能力的要求本来就不高。我们选择放弃一点模型能力，换来的是所有人都能用、能长期用、不用担心哪天突然断了。这个取舍是值得的。

前往官网下载 Qclaw：**https://qclaw.com**

下载安装完成后，打开软件，用手机微信扫码绑定账号，就可以用了。

---

#### 第二步：把 playmore-skills 装进 Qclaw

打开 Qclaw，在对话框里输入下面这句话，直接发送：

```
请你帮我下载 https://github.com/kantdefans-lab/playmore-skills 并安装为 Qclaw 的 skill
```

Qclaw 会自动去 GitHub 下载这个项目，然后把里面的 skill 文件安装到正确的位置。你不需要手动操作任何文件，等它说"安装完成"就行了。

---

#### 第三步：让 Qclaw 能控制 Chrome 浏览器

谷歌学术和知网需要通过浏览器来搜索，所以我们要给 Qclaw 装一个能控制 Chrome 的工具。

在 Qclaw 的对话框里输入：

```
请你帮我安装 chrome-devtools MCP，命令是：qclaw mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

或者如果你熟悉命令行，直接在终端运行：

```bash
qclaw mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

---

#### 第四步：安装谷歌学术和知网的搜索 skill

playmore 依赖两个专门负责搜索的 skill：

- 谷歌学术搜索：[gs-skills](https://github.com/cookjohn/gs-skills)，打开链接，按里面的说明安装
- 知网搜索：[cnki-skills](https://github.com/cookjohn/cnki-skills)，同上

安装方式和第二步一样，在 Qclaw 里让它帮你下载安装就行。

---

#### 第五步：建一个存放笔记的文件夹，填好配置

在 Qclaw 里输入：

```
请帮我在桌面创建一个叫 playmore 的文件夹，然后把 playmore-skills 里的 config.example.json 复制到那里，改名为 config.json
```

然后用记事本或任何文本编辑器打开 `~/Desktop/playmore/config.json`，把里面的关键词换成你自己的研究方向，比如：

```json
{
  "keywords": ["AI 心理健康", "数字经济", "社会资本"],
  "language": "zh"
}
```

---

#### 第六步：每次用之前，先启动 Chrome 调试模式

谷歌学术和知网的搜索需要通过 Chrome 来做，所以每次使用前要先用下面的命令启动 Chrome。

**Windows 用户**，在开始菜单搜索"命令提示符"，打开后粘贴这行命令运行：

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

**macOS 用户**，打开"终端"，粘贴这行命令运行：

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

运行后 Chrome 会打开，保持这个窗口开着，不要关。然后回到 Qclaw 开始用。

---

### 怎么用

#### 搜索文献

在 Qclaw 里输入 `/playmore` 加上你的主题，比如：

```
/playmore AI 心理健康

/playmore 数字经济 劳动力市场

/playmore 社会资本 教育不平等
```

它会自动去四个数据库搜索，去重，打分，AI 点评，然后把笔记存到你桌面的 playmore 文件夹里。整个过程你不需要做任何事，等它跑完就行。

#### 设定时任务，让它自动追踪新文献

如果你有一个长期在跟的研究方向，可以设一个定时任务，让它每隔一段时间自动搜一次：

```
/playmore-schedule add "AI 治疗" from 2026-05-01 to 2026-05-31
```

这样在 5 月份，每次你打开 Qclaw 运行 `/playmore`，它会自动检查有没有到期的任务，有的话直接跑。

管理定时任务的其他指令：

```
/playmore-schedule list          # 查看所有任务
/playmore-schedule run task_001  # 手动触发某个任务
/playmore-schedule edit task_001 # 修改某个任务
/playmore-schedule delete task_001 # 删除某个任务
```

#### 直接运行，检查今天有没有任务

```
/playmore
```

不带任何主题直接运行，它会检查今天有没有到期的定时任务，有的话自动执行。

---

### 笔记长什么样

所有笔记存在 `~/Desktop/playmore/`，按你的研究概念自动分文件夹：

```
~/Desktop/playmore/
├── config.json
├── schedule.json
├── INDEX.md                          # 所有笔记的总索引，自动生成
├── AI×心理健康/
│   ├── 2024_smith_llm-therapy.md
│   └── 2023_wang_chatbot-counseling.md
├── 数字经济×劳动力市场/
│   └── 2023_zhang_platform-work.md
└── ...
```

每篇笔记的内容包括：

```yaml
---
title: 论文标题
authors: [作者列表]
year: 2024
journal: 期刊名
journal_tier: SCI Q1 / CSSCI / 北大核心 / unknown
review: ⭐必读
concept: AI×心理健康
keywords: [关键词]
sources:
  - platform: OpenAlex
    landing_url: 原文页面链接
    pdf_url: PDF 直链（如有）
    doi: 10.xxxx/xxxxx
score: 8
date_added: 2026-04-27
---

## AI 点评
这篇文章是目前 LLM 用于心理咨询领域最系统的综述之一，...

## 核心贡献
...

## 研究方法
...

## 主要结论
...

## 局限性
...

## 创新性
...

## 摘要
...
```

---

### 打分逻辑

| 维度 | 最高分 | 说明 |
|------|--------|------|
| 关键词匹配 | 5 | 标题出现 +2，摘要出现 +1，每个关键词独立计算 |
| 期刊分区 | 4 | Q1=4, Q2=3, Q3=2, CSSCI/北大核心=1 |
| SCI Q3 及以上加权 | 2 | 高分区期刊额外加分 |

总分低于 2 分的论文直接过滤，最终保留得分最高的前 30 篇。

---

### 常见问题

**Q：我完全不懂编程，真的能用吗？**

能。安装过程里最难的操作是"打开命令提示符粘贴一行命令"，其他的都是在 Qclaw 里跟 AI 说话。如果卡住了，直接把报错信息发给 Qclaw 里的 AI，让它帮你解决。

**Q：谷歌学术在国内能搜吗？**

谷歌学术在国内有时候能访问，有时候不行，取决于你的网络环境。如果搜不到，playmore 会自动跳过谷歌学术，用 OpenAlex 和 Semantic Scholar 补充，这两个在国内完全可以访问，而且覆盖面很广。知网需要你有账号（学校一般都有）。

**Q：Qclaw 的免费额度够用吗？**

日常查文献够用。playmore 的设计本来就是尽量少用 AI token，先用规则筛，再用 AI 判，不会无谓地消耗额度。

**Q：笔记存在哪里，会不会丢？**

存在你自己电脑的桌面上，`~/Desktop/playmore/` 文件夹里，都是普通的 Markdown 文本文件，用记事本就能打开。不依赖任何云服务，不会丢。

---

### License

MIT

---

<a id="english"></a>

## English

**playmore-skills** is the first AI-powered academic paper retrieval and analysis system built specifically for social science graduate students and researchers.

No VPN. No coding. No overseas accounts. Just a WeChat-linked phone and you're set.

> **Less literature review, more living.**

### Three Principles

**1. Make AI accessible to every social science student.**
No VPN, no code, no credit card. Every tool and every step in this system was chosen under one constraint: it has to work for someone with zero technical background, sustainably, long-term.

**2. Minimum tokens, maximum results.**
AI usage has a cost. playmore uses rules to filter first, AI to judge second — keeping AI focused on papers that actually matter. Efficient by design.

**3. Let AI do the work.**
Search, dedup, score, review, save notes — fully automated. You type a topic, it handles the rest.

### What it does

- Searches Google Scholar, CNKI, OpenAlex, and Semantic Scholar simultaneously, deduplicating by DOI and title
- Scores papers across three dimensions: keyword relevance + journal tier (SCI/CSSCI) + high-tier boost
- AI classifies each paper: ⭐ Must-read / 📖 Worth reading / 💡 For reference, with a written rationale
- Auto-creates concept folders like `AI×mental-health`, `digital-economy×labor-market`
- Scheduled tasks: set a topic and date range, runs automatically
- Every note records PDF link, DOI, and landing page

### Recommended setup: Qclaw

Qclaw is a domestic AI CLI tool built on WeChat. No VPN, no overseas payment, free tier included. Its model is less capable than Claude or GPT — but playmore only needs retrieval and structured formatting, not complex reasoning. The tradeoff is worth it.

Download: **https://qclaw.com**

### Installation

**Step 1** — Download and install Qclaw, log in with WeChat.

**Step 2** — In Qclaw, send this message:

```
请你帮我下载 https://github.com/kantdefans-lab/playmore-skills 并安装为 Qclaw 的 skill
```

**Step 3** — Install Chrome DevTools MCP:

```bash
qclaw mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

**Step 4** — Install [gs-skills](https://github.com/cookjohn/gs-skills) and [cnki-skills](https://github.com/cookjohn/cnki-skills) following their READMEs.

**Step 5** — Create storage folder and config:

```bash
mkdir ~/Desktop/playmore
cp config.example.json ~/Desktop/playmore/config.json
# Edit config.json with your research keywords
```

**Step 6** — Before each session, start Chrome with remote debugging:

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

### Usage

```bash
# Search a topic
/playmore AI mental health

# Schedule recurring searches
/playmore-schedule add "AI therapy" from 2026-05-01 to 2026-05-31
/playmore-schedule list
/playmore-schedule run task_001

# Check and run today's scheduled tasks
/playmore
```

### Storage structure

```
~/Desktop/playmore/
├── config.json
├── schedule.json
├── INDEX.md
├── AI×心理健康/
│   └── 2024_smith_llm-therapy.md
└── ...
```

### Scoring

| Dimension | Max | Notes |
|-----------|-----|-------|
| Keyword match | 5 | Title +2, abstract +1 per keyword |
| Journal tier | 4 | Q1=4, Q2=3, Q3=2, CSSCI/北大核心=1 |
| SCI Q3+ boost | 2 | Extra weight for high-tier journals |

Papers below score 2 are filtered. Top 30 are kept.

### License

MIT
