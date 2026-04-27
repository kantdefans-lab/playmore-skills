---
name: playmore-schedule
description: Manage scheduled paper search tasks for playmore. Add, list, edit, or delete tasks with date ranges and keywords.
argument-hint: "[add|list|edit|delete|run] ..."
---

# Playmore Schedule

Manage scheduled search tasks stored in `~/Desktop/playmore/schedule.json`.

## Schedule file format

```json
[
  {
    "id": "task_001",
    "keywords": ["AI mental health", "LLM therapy"],
    "date_from": "2026-04-01",
    "date_to": "2026-04-30",
    "active": true,
    "created": "2026-04-24"
  }
]
```

## Commands

### `add` — Add a new task

Parse from $ARGUMENTS:
- Keywords (required)
- Date range: `from YYYY-MM-DD to YYYY-MM-DD` or natural language like `this month`, `April`, `next week`
- Convert natural language dates to absolute YYYY-MM-DD

Generate a new unique id (`task_NNN`), append to schedule.json, confirm to user.

### `list` — Show all tasks

Read schedule.json and display:

```
定时任务列表：

[task_001] ✅ 进行中
  关键词：AI mental health, LLM therapy
  时间段：2026-04-01 → 2026-04-30

[task_002] ⏸ 已暂停
  ...
```

### `edit` — Modify a task

Parse task id and new values from $ARGUMENTS. Update the matching entry in schedule.json.

### `delete` — Remove a task

Parse task id from $ARGUMENTS. Remove from schedule.json. Confirm deletion.

### `run` — Force run a specific task now

Parse task id. Load its keywords and run `/playmore {keywords}`.

## Auto-run check (called by playmore with no arguments)

Read schedule.json. For each active task where today falls within `date_from`–`date_to`:
- Run `/playmore {task.keywords joined by space}`
- Report which task triggered

If no tasks match today, say: "今天没有定时任务。直接输入关键词开始搜索，或用 `/playmore-schedule add` 添加定时任务。"

## If schedule.json does not exist

Create it as an empty array `[]` and prompt the user to add a task.
