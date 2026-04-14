# Design: Team Leaderboard

## Architecture

```
Admin API Key (one-time setup by org admin)
     │
     ▼
Cron Script (single machine, daily)
     │  GET /v1/organizations/usage_report/claude_code
     │  Paginate through all users
     │  Write JSON to data/<YYYY-MM-DD>.json
     ▼
Git Repo (this repo)
     ├─ data/
     │    ├─ 2026-04-14.json   ← full team data for that day
     │    ├─ 2026-04-13.json
     │    └─ ...
     └─ dashboard/
          └─ index.html        ← static page, reads data/*.json, renders leaderboard
```

**Key property**: Only 1 cron job on 1 machine. No per-developer setup needed.

## Leaderboard Dimensions

Based on actual API fields available:

| Leaderboard | Calculation | Measures |
|-------------|-------------|----------|
| **Token Burn** | Σ(input + output + cache_read + cache_creation) | Total consumption (Meta-style) |
| **Productivity** | lines_of_code.added + commits + PRs | Actual code output |
| **Efficiency** | lines_added / total_tokens | Code output per token |
| **Tool Acceptance** | accepted / (accepted + rejected) | Prompt quality / human-AI collaboration |
| **Consistency** | Days with activity in period | Usage habit |
| **Cost Effectiveness** | (lines_added + commits × 50) / estimated_cost | Output per dollar |

## Dashboard Features (Planned)

### Main View
- Leaderboard table: rank, name, tokens, cost, lines of code, cache hit rate
- Time range selector: 7d / 30d / custom
- Toggle between leaderboard dimensions

### Trend View
- Per-user token consumption over time (line chart)
- Team aggregate trend

### Model View
- Token distribution by model (pie/bar chart)
- Per-user model preference

## Tech Stack (Proposed)

- **Data collection**: Shell script + cron (or GitHub Actions scheduled workflow)
- **Storage**: JSON files in git (simple, auditable, no DB needed)
- **Dashboard**: Static HTML + JS (vanilla or lightweight framework)
- **Hosting**: GitHub Pages or internal GitLab Pages

## Prerequisites

1. Org admin creates Admin API Key at Console → Settings → Admin Keys
2. Store key securely (env var or CI secret, NOT in repo)
3. Set up daily cron or GitHub Actions schedule

## Privacy Considerations

- API returns only aggregated metrics (token counts, lines of code, etc.)
- No conversation content, no code content, no prompts
- Email addresses are included — dashboard may want to map to display names
- Data stored in git is visible to repo collaborators

## Rejected Alternatives

| Approach | Why Rejected |
|----------|-------------|
| ccusage local collection per developer | Requires N-machine setup, relies on individual compliance |
| OpenTelemetry integration | Overkill for leaderboard use case, complex setup |
| Anthropic Console manual export | Not automatable, no per-user breakdown for Team plan |
| Scraping Console UI | Fragile, against ToS |
