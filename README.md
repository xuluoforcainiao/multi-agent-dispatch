# Multi-Agent Dispatch

> QoderWork skill for coordinating tasks between multiple AI agents via a shared DingTalk AI spreadsheet.
>
> **Protocol version**: v1.1

## What is this?

This skill enables two or more AI agents (e.g., `wukong` and `qoderwork`) to collaborate on tasks without HTTP APIs, webhooks, or direct messaging. Agents communicate exclusively through a **shared DingTalk AI spreadsheet** that acts as a persistent message bus and state machine.

## Architecture

```
Agent A (wukong) ──writes task──▶ Shared Spreadsheet ──polls──▶ Agent B (qoderwork)
         │                                                            │
         │◀────────reads result───────────────────────────────────────┘
```

**Key principle**: No HTTP calls. No group chat messages. Just a spreadsheet both agents can read and write.

## Core Features

- **Zero-HTTP Communication** — Agents communicate via shared DingTalk AI spreadsheet, no public endpoints required
- **Auto-Shutdown Polling** — Cron job auto-disables after all tasks complete, saving tokens
- **Structured Task Queue** — 12-field spreadsheet with status machine (`pending → in_progress → done/failed → confirmed`)
- **Human-in-the-Loop** — Only one manual step: user enables the cron job when tasks are pending
- **Race-Condition Safe** — Re-queries spreadsheet before auto-shutdown to avoid missing new tasks

## Quick Start

### Prerequisites

- Both agents have access to the same DingTalk AI spreadsheet
- Agent B (qoderwork) has a QoderWork cron job configured

### 1. Create the Spreadsheet

Use DingTalk AI spreadsheet with these fields:

| Field | Type | Purpose |
|-------|------|---------|
| 任务标题 | primaryDoc | Task summary |
| 任务ID | text | Unique ID (`{from}_{timestamp}_{seq}`) |
| 任务类型 | singleSelect | data_analysis, report_generation, etc. |
| 任务描述 | richText | Detailed instructions |
| 优先级 | singleSelect | P0 / P1 / P2 |
| 状态 | singleSelect | pending → in_progress → done/failed → confirmed |
| 发起方 | singleSelect | wukong / qoderwork |
| 接收方 | singleSelect | wukong / qoderwork |
| 回调地址 | url | Optional (unused in v1.1) |
| 结果摘要 | richText | Execution result |
| 结果附件 | attachment | Result files |
| 完成时间 | date | Completion timestamp |

### 2. Configure the Cron Job

| Property | Value |
|----------|-------|
| Name | `QW悟空协调` |
| Interval | Every 60 seconds |
| Default State | **Disabled** |

### 3. Run a Task

**Step 1** — Agent A writes a task row with `status = pending`

**Step 2** — Agent A notifies user in group: "@user 新任务已下发，请打开小Q定时任务"

**Step 3** — User enables cron: "打开定时任务"

**Step 4** — Agent B auto-polls, executes, and reports results

**Step 5** — Agent B auto-shuts down and IMs user: "全部任务已完成，定时任务已自动关闭"

**Step 6** — User notifies Agent A: "小Q任务做完了"

**Step 7** — Agent A confirms by updating status to `confirmed`

## Protocol Versions

### v1.1 (Current)

- **Auto-shutdown**: Agent B auto-disables cron after all tasks complete
- **Single human touch point**: User only needs to enable (no manual disable)

### v1.0

- Manual on/off switch: User had to both enable and disable the cron job

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill documentation with full protocol |
| `reference.md` | Field IDs, option IDs, MCP tool examples |
| `README.md` | This file |

## Known Limitations

1. **No DingTalk group chat support** for qoderwork (direct messages only)
2. **No HTTP callbacks** — neither agent exposes public endpoints
3. **Race condition window**: ~1-60 seconds between final query and auto-shutdown (acceptable for most use cases)

## Future Improvements

- [ ] Expand task type enum based on real collaboration scenarios
- [ ] Auto-upload result files to DingTalk cloud drive
- [ ] Support multi-agent dispatch (3+ agents)

## License

MIT
