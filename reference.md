# Reference: Multi-Agent Dispatch Configuration

## Cron Job

| Property | Value |
|----------|-------|
| Cron Name | QW悟空协调 |
| Cron ID | `955c8b4b-4727-44c2-b224-00c069e1fc83` |
| Schedule | Every 60 seconds (60000ms) |
| Default State | **Disabled** |
| Enable Command | `qoder_cron` action `enable`, jobId `955c8b4b-4727-44c2-b224-00c069e1fc83` |
| Disable Command | Auto-shutdown by Agent B after all tasks done. Manual disable via `qoder_cron` action `disable` for emergency stop only. |

## Spreadsheet IDs

| Property | Value |
|----------|-------|
| Base Name | Agent协作任务队列 |
| Base ID | `dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N` |
| Table Name | 任务队列 |
| Table ID | `MBsdvCs` |
| Spreadsheet URL | `https://docs.dingtalk.com/i/nodes/dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N` |
| DingTalk Group | 悟空-小Q 协作群 |
| Group OpenConversationId | `cidKP24pr3OhJttJ6bsKOiGjw==` |

## Field ID Mapping

| Field Name | fieldId | Type |
|------------|---------|------|
| 任务标题 | `McSTWbC` | primaryDoc |
| 任务ID | `Zc5JkYX` | text |
| 任务类型 | `PHtVlfB` | singleSelect |
| 任务描述 | `gBrSo8a` | richText |
| 优先级 | `SBw8yqM` | singleSelect |
| 状态 | `kQxGXyR` | singleSelect |
| 发起方 | `aCh5Hq7` | singleSelect |
| 接收方 | `N131pAt` | singleSelect |
| 回调地址 | `wIKjYA0` | url |
| 结果摘要 | `UjQVwXU` | richText |
| 结果附件 | `kYjuQWD` | attachment |
| 完成时间 | `ueIyVzW` | date |

## SingleSelect Option ID Mapping

### 任务类型 (PHtVlfB)

| Option Name | Option ID |
|-------------|-----------|
| data_collection | `9BqZN2W0fU` |
| report_generation | `x6k0onzXBj` |
| data_analysis | `tuMkAo0TPU` |
| file_processing | `Csiy74NQyo` |
| notification | `fmdhzNWoSR` |
| ping | `qBZcsPDEqB` |

### 优先级 (SBw8yqM)

| Option Name | Option ID |
|-------------|-----------|
| P0 | `2JwI0HaaZU` |
| P1 | `ASAUAr3xdB` |
| P2 | `mmr0AqXQTo` |

### 状态 (kQxGXyR)

| Option Name | Option ID |
|-------------|-----------|
| pending | `eoVGCd5Ojj` |
| in_progress | `jLeuyisFU4` |
| done | `I4SdTy7s51` |
| failed | `GAeTagt8gm` |
| confirmed | `2SmrWserpE` |

### 发起方 (aCh5Hq7)

| Option Name | Option ID |
|-------------|-----------|
| wukong | `GnD2D38KG1` |
| qoderwork | `iOYQm6f7LU` |

### 接收方 (N131pAt)

| Option Name | Option ID |
|-------------|-----------|
| wukong | `2TJw1DX6lA` |
| qoderwork | `j7dEgArVdQ` |

## MCP Tool Examples

### Query Pending Tasks

Tool: `mcp__钉钉 AI 表格__query_records`

```json
{
  "baseId": "dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N",
  "tableId": "MBsdvCs",
  "filters": {
    "operator": "and",
    "operands": [
      {
        "operator": "eq",
        "operands": ["N131pAt", "j7dEgArVdQ"]
      },
      {
        "operator": "eq",
        "operands": ["kQxGXyR", "eoVGCd5Ojj"]
      }
    ]
  },
  "limit": 10
}
```

### Update Task Status

Tool: `mcp__钉钉 AI 表格__update_records`

```json
{
  "baseId": "dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N",
  "tableId": "MBsdvCs",
  "records": [
    {
      "recordId": "RECORD_ID_HERE",
      "cells": {
        "kQxGXyR": {"id": "jLeuyisFU4", "name": "in_progress"},
        "ueIyVzW": "2026-05-17 22:12"
      }
    }
  ]
}
```

### Mark Task as Done

```json
{
  "baseId": "dQPGYqjpJYZnRbNYCZEdqbn68akx1Z5N",
  "tableId": "MBsdvCs",
  "records": [
    {
      "recordId": "RECORD_ID_HERE",
      "cells": {
        "kQxGXyR": {"id": "I4SdTy7s51", "name": "done"},
        "UjQVwXU": {"markdown": "**Result: success**"},
        "ueIyVzW": "2026-05-17 22:12"
      }
    }
  ]
}
```

## Field Value Formats

### richText
```json
{"markdown": "**Bold text**\nNormal text\n"}
```

### date
Format: `YYYY-MM-DD HH:mm` (e.g., `2026-05-17 22:12`)

### singleSelect
```json
{"id": "OPTION_ID", "name": "OPTION_NAME"}
```

### url
```json
{"text": "Display text", "link": "https://..."}
```

### attachment
```json
[{"fileToken": "ft_xxx"}]
```
