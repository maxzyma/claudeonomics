# Background: Why Claudeonomics

## Origin

Meta internally built a leaderboard called "Claudeonomics" tracking 85,000+ employees' AI token usage, ranking top 250 "super users" with gamified titles. In 30 days, 60 trillion tokens were consumed; the #1 user burned 281 billion tokens (avg 9.36B/day). Meta shut it down after data leaked externally.

Source: [创新观察局 2026-04-09](https://mp.weixin.qq.com/s/Gzq2KAts-PwM2JlhyFrEdg)

## Our Context

- Team uses Claude Code **Team subscription** (not API pay-as-you-go)
- Goal: build a similar team leaderboard feasible under Team subscription constraints
- Project name "claudeonomics" is a nod to Meta's original, no GitHub name conflict exists

## Key Insight from Research

Initially considered a local-log-collection approach (each developer runs `ccusage` and reports data). This was rejected because:

1. Relies on every team member to configure cron jobs — fragile
2. Data completeness depends on individual compliance
3. Higher maintenance cost (N machines vs 1)

Instead, discovered **Anthropic's official Claude Code Analytics API** — a centralized, admin-level API that provides per-user daily metrics without any client-side setup.
