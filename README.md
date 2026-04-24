# playmore-skills

[English](#english) | [中文](#中文)

---

<a id="english"></a>

## English

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for cross-platform academic paper tracking. Search Google Scholar, CNKI, OpenAlex, and Semantic Scholar — score, review, and save structured notes organized by concept.

### Features

- Cross-source search: Google Scholar, CNKI, OpenAlex, Semantic Scholar
- Three-dimensional scoring: keyword relevance + journal tier + SCI Q3+ boost
- AI review with three levels: ⭐ Must-read / 📖 Worth reading / 💡 For reference
- Concept-based note organization: dynamic `concept×concept` folder creation
- Scheduled tasks: define date ranges and keywords, run automatically or on demand
- Full source tracking: PDF links, DOI, landing page recorded per paper

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI (or compatible: Codex, Qclaw)
- Chrome with remote debugging (for Google Scholar and CNKI)
- [gs-skills](https://github.com/cookjohn/gs-skills) installed
- [cnki-skills](https://github.com/cookjohn/cnki-skills) installed
- OpenAlex and Semantic Scholar: no API key needed

### Skills

| Skill | Description | Invocation |
|-------|-------------|------------|
| `playmore` | Main entry — routes to sub-skills or checks scheduled tasks | `/playmore AI mental health` |
| `playmore-fetch` | Search all four sources, deduplicate by DOI/title | called by playmore |
| `playmore-score` | Score papers: keyword + journal tier + SCI Q3+ boost | called by playmore |
| `playmore-review` | AI classification and commentary | called by playmore |
| `playmore-notes` | Generate structured Markdown notes, save by concept folder | called by playmore |
| `playmore-schedule` | Manage scheduled search tasks | `/playmore-schedule add ...` |

### Installation

#### 1. Install Chrome DevTools MCP

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. Install gs-skills and cnki-skills

Follow their respective README instructions.

#### 3. Install playmore-skills

**Claude Code:**
```bash
git clone https://github.com/kantdefans-lab/playmore-skills.git
cp -r playmore-skills/skills/* ~/.claude/skills/
```

**Codex:**
```bash
cp -r playmore-skills/skills/* ~/.codex/skills/
```

**Qclaw:**
```bash
cp -r playmore-skills/skills/* ~/.qclaw/skills/
```

#### 4. Create storage directory and config

```bash
mkdir ~/Desktop/playmore
cp config.example.json ~/Desktop/playmore/config.json
# Edit config.json with your keywords
```

#### 5. Start Chrome with remote debugging

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

### Usage

```bash
# Manual search
/playmore AI mental health

# Manage scheduled tasks
/playmore-schedule add "AI therapy" from 2026-05-01 to 2026-05-31
/playmore-schedule list
/playmore-schedule edit task_001
/playmore-schedule delete task_001
/playmore-schedule run task_001

# Check and run today's scheduled tasks
/playmore
```

### Storage structure

```
~/Desktop/playmore/
├── config.json
├── schedule.json
├── INDEX.md                        # Auto-generated index
├── AI×心理健康/
│   └── 2024_smith_llm-therapy.md
├── 深度学习×情绪识别/
│   └── 2023_zhang_emotion-detection.md
└── ...
```

### Note format

```yaml
---
title: ...
authors: [...]
year: ...
journal: ...
journal_tier: SCI Q1 / Q2 / Q3 / CSSCI / 北大核心 / unknown
review: ⭐必读 / 📖可读 / 💡了解
concept: AI×心理健康
keywords: [...]
sources:
  - platform: OpenAlex
    landing_url: ...
    pdf_url: ...
    doi: ...
score: 8
date_added: 2026-04-24
---
```

Followed by: AI点评, 核心贡献, 研究方法, 主要结论, 局限性, 创新性, 摘要.

### Scoring

| Dimension | Max |
|-----------|-----|
| Keyword match (title +2, abstract +1 per keyword) | 5 |
| Journal tier (Q1=4, Q2=3, Q3=2, CSSCI/北大核心=1) | 4 |
| SCI Q3+ keyword boost | 2 |

Papers scoring below 2 are excluded. Top 30 are kept.

### License

MIT

---

<a id="中文"></a>

## 中文

跨平台学术论文追踪 skill 套件，适用于 Claude Code / Codex / Qclaw。

同时搜索谷歌学术、知网、OpenAlex、Semantic Scholar，自动打分、AI 点评，按概念分层保存结构化笔记。

### 功能

- 四源跨平台搜索，按 DOI/标题去重
- 三维打分：关键词相关度 + 期刊分区 + SCI Q3 以上加权
- AI 三级点评：⭐必读 / 📖可读 / 💡了解
- 概念分层存储：动态识别 `概念×概念` 并新建文件夹
- 定时任务：设定日期范围和关键词，自动或手动触发
- 完整来源记录：PDF 链接、DOI、原文页面

### 前置要求

- Claude Code CLI（或兼容工具：Codex、Qclaw）
- Chrome 开启远程调试（用于谷歌学术和知网）
- 已安装 [gs-skills](https://github.com/cookjohn/gs-skills)
- 已安装 [cnki-skills](https://github.com/cookjohn/cnki-skills)
- OpenAlex 和 Semantic Scholar 无需 API key

### 安装

#### 1. 安装 Chrome DevTools MCP

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. 安装 gs-skills 和 cnki-skills

参照各自 README。

#### 3. 安装 playmore-skills

**Claude Code：**
```bash
git clone https://github.com/kantdefans-lab/playmore-skills.git
cp -r playmore-skills/skills/* ~/.claude/skills/
```

**Codex：**
```bash
cp -r playmore-skills/skills/* ~/.codex/skills/
```

**Qclaw：**
```bash
cp -r playmore-skills/skills/* ~/.qclaw/skills/
```

#### 4. 创建存储目录和配置文件

```bash
mkdir ~/Desktop/playmore
cp config.example.json ~/Desktop/playmore/config.json
# 编辑 config.json，填入你的关键词
```

#### 5. 启动 Chrome 远程调试

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

### 使用方式

```bash
/playmore AI 心理健康
/playmore-schedule add "AI 治疗" from 2026-05-01 to 2026-05-31
/playmore-schedule list
/playmore
```

### License

MIT
