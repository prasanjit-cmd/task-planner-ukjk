# Task Planner

> A full-stack Google Ads operations agent built around quota-aware ingestion, analysis, recommendation triage, approval-gated mutation execution, durable audit history, RAG memory, and a Mission Control dashboard. The architecture uses Google Ads REST as the primary integration, SQLite for structured state and analytics, a vector database for account memory and historical explanations, custom dashboard APIs for read models, and OpenClaw chat surfaces for conversational control and approvals. The plan favors explicit guardrails, bounded mutation batches, daily and intra-day syncs, durable change history, and granular skills that can be composed into scheduled workflows or manual operator actions.

## Structure
- `SOUL.md` — Agent personality, rules, and mission
- `skills/` — Agent skills (one directory per skill)
- `.openclaw/config.yml` — Runtime configuration

## Deploy
```bash
openclaw deploy .
```

Built with [Ruh.ai](https://ruh.ai)