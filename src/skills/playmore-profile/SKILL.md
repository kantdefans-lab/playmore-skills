---
name: playmore-profile
description: Manage the researcher profile for playmore. Reads and updates profile.md to personalize paper scoring and review over time. Called automatically by playmore.
argument-hint: "[init|update|show]"
---

# Playmore Profile — Researcher Memory

Manages `~/Desktop/playmore/profile.md` — a persistent memory file that grows smarter with every search session.

## Profile file location

`~/Desktop/playmore/profile.md`

---

## Command: `init`

Called when no profile.md exists and the user wants to set up their research context.

Ask the user (in a single conversational message, not a form):

```
你好！在开始之前，简单跟我说说你在研究什么吧——方向、关注的问题、或者最近在看什么都行，随便说。
```

From their response, extract:
- Research field and subfield
- Core research questions or topics
- Keywords (explicit or implied)
- Negative keywords (topics they're NOT interested in)
- Language preference (Chinese / English / both)

Then create `~/Desktop/playmore/profile.md` using the template below.

---

## Command: `update`

Called automatically by `/playmore` after each search session completes.

Receives from playmore:
- `search_keywords`: what the user searched for
- `papers_found`: total papers before filtering
- `papers_kept`: papers after scoring
- `must_read_count`: number of ⭐必读 papers
- `concepts_found`: list of concept labels assigned
- `top_journals`: journals appearing in must-read papers
- `top_keywords`: keywords extracted from must-read paper titles/abstracts

Update `profile.md` as follows:

1. **Increment** `total_searches` in frontmatter
2. **Update** `last_updated` to today
3. **Merge keywords**: for each keyword in `top_keywords`, if already in the keyword table, increase weight by 1 (max 10); if new, add with weight 2
4. **Update search history**: prepend a new row to the search history table (keep last 20 rows)
5. **Update concept map**: for each concept in `concepts_found`, increment paper count or add new row
6. **Update reading preferences**: if must-read papers cluster around certain journals or methods, note it
7. **Regenerate researcher portrait**: rewrite the 研究者画像 section based on the full updated history (2-3 sentences, factual, no fluff)
8. **Append to 研究方向演变**: one line with today's date and a brief note on what this search reveals about the user's evolving focus

---

## Command: `show`

Display the current profile in a readable format. If no profile exists, say so and suggest running `/playmore-profile init`.

---

## Profile file template

```markdown
---
last_updated: {YYYY-MM-DD}
total_searches: 0
---

## 研究者画像

{1-2 sentences describing the researcher's field, focus, and approach. Updated automatically after each search. Example: "社会学方向研究者，主要关注数字经济对劳动力市场的结构性影响，尤其是平台经济与低收入群体就业。研究偏向实证定量方法，中英文文献并重。"}

---

## 关键词权重

权重越高，打分时加成越大（最高 10）。每次搜索后自动更新。

| 关键词 | 权重 | 最近出现 |
|--------|------|----------|
| {keyword} | {weight} | {YYYY-MM-DD} |

## 负向关键词

{comma-separated list of topics to exclude, or "暂无"}

---

## 阅读偏好

- 期刊偏好：{journals that appear most in must-read papers, or "待积累"}
- 方法偏好：{quantitative / qualitative / mixed, inferred from abstracts, or "待积累"}
- 语言偏好：{Chinese / English / both}

---

## 已探索概念

| 概念 | 论文数 | 最近更新 |
|------|--------|----------|
| {concept} | {count} | {YYYY-MM-DD} |

---

## 搜索历史

| 日期 | 关键词 | 找到 | 保留 | 必读 |
|------|--------|------|------|------|
| {date} | {keywords} | {found} | {kept} | {must_read} |

---

## 研究方向演变

{One line per search session, appended automatically}
- {YYYY-MM-DD}：{brief note on what this search reveals}
```

---

## How profile personalizes scoring

When `/playmore-score` reads the profile, it uses keyword weights as multipliers:

- Profile keyword weight 8–10 → title match gives +3 pts (instead of +2)
- Profile keyword weight 5–7 → normal scoring
- Profile keyword weight 1–4 → title match gives +1 pt

This means papers in the user's core research area score higher over time without any manual configuration.

---

## Privacy note

The profile is stored only on the user's local machine at `~/Desktop/playmore/profile.md`. It is never sent anywhere. The user can open, edit, or delete it at any time.
