# Data Source: Claude Code Analytics API

## API Endpoint

```
GET /v1/organizations/usage_report/claude_code
```

- **Auth**: Admin API Key (`sk-ant-admin-...`), provisioned by org admin at Console → Settings → Admin Keys
- **Docs**: https://platform.claude.com/docs/en/api/claude-code-analytics-api
- **Cost**: Free for all organizations with Admin API access
- **Data freshness**: ~1 hour delay
- **Aggregation**: Daily, per user
- **Pagination**: Cursor-based, up to 1000 records per page

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `starting_at` | string | Yes | UTC date (YYYY-MM-DD), returns metrics for this single day |
| `limit` | integer | No | Records per page (default: 20, max: 1000) |
| `page` | string | No | Cursor token from previous response's `next_page` |

## Response Fields (per user per day)

### Dimensions

| Field | Description |
|-------|-------------|
| `date` | RFC 3339 UTC timestamp |
| `actor.type` | `user_actor` (email) or `api_actor` (API key name) |
| `actor.email_address` | User email (when type = user_actor) |
| `organization_id` | Org UUID |
| `customer_type` | `api` or `subscription` (Pro/Team) — **Team subscription users are included** |
| `terminal_type` | vscode / iTerm.app / tmux / etc. |

### Core Metrics

| Field | Description |
|-------|-------------|
| `num_sessions` | Distinct CC sessions initiated |
| `lines_of_code.added` | Lines added across all files |
| `lines_of_code.removed` | Lines removed across all files |
| `commits_by_claude_code` | Git commits created through CC |
| `pull_requests_by_claude_code` | PRs created through CC |

### Tool Actions

| Field | Description |
|-------|-------------|
| `edit_tool.accepted / rejected` | Edit tool acceptance/rejection count |
| `write_tool.accepted / rejected` | Write tool acceptance/rejection count |
| `multi_edit_tool.accepted / rejected` | Multi-edit tool acceptance/rejection count |
| `notebook_edit_tool.accepted / rejected` | NotebookEdit acceptance/rejection count |

### Model Breakdown (array, per model)

| Field | Description |
|-------|-------------|
| `model` | Model ID (e.g., `claude-opus-4-6`) |
| `tokens.input` | Input token count |
| `tokens.output` | Output token count |
| `tokens.cache_read` | Cache read tokens |
| `tokens.cache_creation` | Cache creation tokens |
| `estimated_cost.amount` | Cost in **cents** USD |
| `estimated_cost.currency` | Always `USD` |

## Example Response

```json
{
  "data": [
    {
      "date": "2026-04-14T00:00:00Z",
      "actor": {
        "type": "user_actor",
        "email_address": "developer@company.com"
      },
      "organization_id": "dc9f6c26-...",
      "customer_type": "subscription",
      "terminal_type": "vscode",
      "core_metrics": {
        "num_sessions": 5,
        "lines_of_code": { "added": 1543, "removed": 892 },
        "commits_by_claude_code": 12,
        "pull_requests_by_claude_code": 2
      },
      "tool_actions": {
        "edit_tool": { "accepted": 45, "rejected": 5 },
        "multi_edit_tool": { "accepted": 12, "rejected": 2 },
        "write_tool": { "accepted": 8, "rejected": 1 },
        "notebook_edit_tool": { "accepted": 3, "rejected": 0 }
      },
      "model_breakdown": [
        {
          "model": "claude-opus-4-6",
          "tokens": {
            "input": 100000,
            "output": 35000,
            "cache_read": 10000,
            "cache_creation": 5000
          },
          "estimated_cost": { "currency": "USD", "amount": 1025 }
        }
      ]
    }
  ],
  "has_more": false,
  "next_page": null
}
```

## Supplementary Data Source: Usage and Cost API

For organization-wide (non-per-user) cost reconciliation:

```
GET /v1/organizations/usage_report/messages  — token consumption by model/workspace
GET /v1/organizations/cost_report            — cost breakdown in USD
```

Docs: https://platform.claude.com/docs/en/api/usage-cost-api
