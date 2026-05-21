---
name: multi-agent-dispatch
description: Coordinate tasks between multiple AI agents (wukong and qoderwork) using a shared DingTalk AI spreadsheet as the message bus. Use when dispatching tasks across agents, checking pending agent tasks, or setting up inter-agent collaboration workflows.
---

# Multi-Agent Task Dispatch

## Overview

This skill defines the protocol for task dispatch and collaboration between multiple AI agents using a shared DingTalk AI spreadsheet as the communication backbone.

**Key principle**: Agents do not communicate via HTTP API or group chat. They communicate exclusively through a shared spreadsheet where one agent writes tasks and the other polls and executes them.

## Architecture

```
Agent A (wukong) ──writes task──▶ Shared Spreadsheet ──polls──▶ Agent B (qoderwork)
         │                                                            │
         │◀────────reads result───────────────────────────────────────┘
```

**Human-in-the-loop**: Agent B (qoderwork) can be triggered manually or via a scheduled cron job (disabled by default to save tokens).

## Polling Mechanism

A QoderWork cron job `QW悟空协调` (ID: 955c8b4b-4727-44c2-b224-00c069e1fc83) runs every 1 minute to auto-detect and execute pending tasks.

| Property | Value |
|----------|-------|
| Cron Name | QW悟空协调 |
| Cron ID | 955c8b4b-4727-44c2-b224-00c069e1fc83 |
| Interval | Every 60 seconds |
| Default State | **Disabled** |

### On/Off Switch Protocol

To avoid idle token consumption, the cron job is disabled by default and must be explicitly enabled when tasks are pending.

**Turn ON** (when Agent A dispatches tasks):
1. Agent A writes tasks to the spreadsheet
2. Agent A notifies in the group: "@浩正 新任务已下发，请打开小Q的定时任务"
3. User enables the cron job via QoderWork chat

**Task Execution** (Agent B cron auto-polls):
1. Cron job queries spreadsheet every 60 seconds for pending tasks
2. For each pending task: claim (in_progress) → execute → report (done/failed)
3. After ALL tasks in the batch are processed: Agent B performs **auto-shutdown**:
   - Re-queries spreadsheet to confirm no pending or in_progress tasks remain
   - Calls `qoder_cron disable` to auto-disable itself
   - Sends IM notification to user: "全部任务已完成，定时任务已自动关闭"

**Turn OFF** (auto-shutdown, no manual step):
1. Agent B auto-disables the cron job after confirming all tasks are done/failed
2. Agent B sends IM notification: "全部任务已完成，定时任务已自动关闭"
3. User notifies Agent A in the group: "小Q任务做完了"
4. Agent A updates task status to `confirmed`

**Boundary rule**: Do NOT auto-shutdown when the first task finishes. Wait until every single task in the batch is `done` or `failed`. Re-query the spreadsheet before disabling to avoid race conditions with new tasks.

**Enable Command** (user to qoderwork):
- "打开定时任务" or "启用 QW悟空协调"

**Disable** is now automatic — Agent B auto-shuts down after all tasks are completed. User only needs to disable manually as an emergency stop.

## Spreadsheet Configuration

| Property | Value |
|----------|-------|
| Base Name | Agent协作任务队列 |
| Base ID | dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N |
| Table Name | 任务队列 |
| Table ID | MBsdvCs |
| Spreadsheet URL | https://docs.dingtalk.com/i/nodes/dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N |

## Task Protocol

### Field Schema

The spreadsheet has 12 custom fields plus 4 system fields:

| Field | Type | Purpose |
|-------|------|---------|
| 任务标题 | primaryDoc | Task one-line summary |
| 任务ID | text | Unique identifier |
| 任务类型 | singleSelect | Task category |
| 任务描述 | richText | Detailed description |
| 优先级 | singleSelect | P0 / P1 / P2 |
| 状态 | singleSelect | Task lifecycle state |
| 发起方 | singleSelect | wukong / qoderwork |
| 接收方 | singleSelect | wukong / qoderwork |
| 回调地址 | url | Optional callback URL (currently unused) |
| 结果摘要 | richText | Execution result |
| 结果附件 | attachment | Result files |
| 完成时间 | date | Completion timestamp |

System fields (auto-managed): 创建时间, 最后编辑时间, 创建人, 最后编辑人.

### Task Types

- `data_collection` — Data collection
- `report_generation` — Report generation
- `data_analysis` — Data analysis
- `file_processing` — File processing
- `notification` — Notification
- `ping` — Connectivity test

### Status Flow

```
pending → in_progress → done / failed → confirmed
```

### Task ID Format

```
{from}_{timestamp}_{seq}
```

Examples: `wk_20260518_001` (wukong), `qq_20260518_001` (qoderwork)

## Operation Flow (v1.1)

### 1. Agent A Dispatches Task

1. Create a new row in the spreadsheet
2. Set status = `pending`
3. Set 接收方 to the target agent
4. Fill in task details (ID, type, description, priority)

### 2. Agent A Notifies User

Agent A sends group message: "@浩正 新任务已下发，请打开小Q定时任务"

### 3. User Enables Agent B

User manually enables the cron job via QoderWork chat: "打开定时任务"

### 4. Agent B Auto-Polls and Executes

1. Cron queries spreadsheet every 60 seconds for pending tasks
2. For each pending task: claim (`in_progress`) → execute → report (`done`/`failed`)
3. After ALL tasks are processed: Agent B auto-shutdowns

### 5. Agent B Auto-Shuts Down

1. Re-queries spreadsheet to confirm no `pending` or `in_progress` tasks remain
2. Calls `qoder_cron disable` to stop itself
3. Sends IM to user: "全部任务已完成，定时任务已自动关闭"

### 6. User Notifies Agent A

User sends group message: "小Q任务做完了"

### 7. Agent A Confirms

1. Polls spreadsheet for updated status
2. Reads results
3. Updates status to `confirmed`

**Human touch points**: Only steps 3 (enable) and 6 (notify) require human action. Step 5 is fully automated.

## Querying Pending Tasks

Use MCP tool `mcp__钉钉 AI 表格__query_records`:

- baseId: `dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N`
- tableId: `MBsdvCs`
- Filter: 接收方 = `qoderwork` AND 状态 = `pending`

For exact field IDs and option IDs, see [reference.md](reference.md).

## Updating Task Status

Use MCP tool `mcp__钉钉 AI 表格__update_records`:

- baseId: `dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N`
- tableId: `MBsdvCs`
- Pass recordId and cells to update

For singleSelect fields, pass option objects like `{"id": "OPTION_ID", "name": "OPTION_NAME"}`.

## Known Limitations

1. **No DingTalk group chat**: qoderwork does not support DingTalk group chat (only direct messages). Group notifications are one-way (Agent A can send, qoderwork cannot receive).
2. **No HTTP callbacks**: Neither agent exposes public HTTP endpoints.
3. **Auto-shutdown**: The cron auto-disables after all tasks are completed. Manual disable is only needed for emergency stop.

## Future Improvements

1. Expand task type enum based on real collaboration scenarios
2. Auto-upload result files to DingTalk cloud drive

## Additional Resources

- For complete field IDs and option IDs, see [reference.md](reference.md)
