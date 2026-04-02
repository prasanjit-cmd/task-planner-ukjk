# Task Planner

> A full-stack Google Ads operations agent built around quota-aware ingestion, analysis, recommendation triage, approval-gated mutation execution, durable audit history, RAG memory, and a Mission Control dashboard. The architecture uses Google Ads REST as the primary integration, SQLite for structured state and analytics, a vector database for account memory and historical explanations, custom dashboard APIs for read models, and OpenClaw chat surfaces for conversational control and approvals. The plan favors explicit guardrails, bounded mutation batches, daily and intra-day syncs, durable change history, and granular skills that can be composed into scheduled workflows or manual operator actions.

## Rules

## Skills
- **Google Ads Auth**: Manages OAuth token refresh, developer token validation, account header construction, and authenticated request preparation for Google Ads REST calls.
- **Google Ads Account Discovery**: Discovers accessible customer accounts and manager relationships, then records which accounts are available for management.
- **Google Ads Structure Sync**: Syncs account structure including budgets, campaigns, ad groups, and keywords so the dashboard and action workflows operate on current entity state.
- **Google Ads Metrics Sync**: Pulls daily performance metrics at account, campaign, ad group, keyword, and scoped search-term grains for historical analysis and dashboarding.
- **Google Ads Recommendation Sync**: Fetches Google-native recommendations, normalizes them into local categories, stores impact estimates, and maintains lifecycle state for review.
- **Google Ads Anomaly Analysis**: Computes pacing status, detects spikes and drops versus trailing baselines or thresholds, and creates structured alerts with explanations.
- **Google Ads Action Planner**: Combines Google recommendations, local heuristics, account rules, and past outcomes to generate approval-ready optimization proposals.
- **Google Ads Mutation Executor**: Executes approved Google Ads changes through bounded mutate requests with before-state capture, partial-failure handling, and full audit logging.
- **Google Ads Outcome Evaluator**: Measures the observed performance impact of executed actions over configured lookback windows and records learning signals for future planning.
- **Google Ads Dashboard API**: Serves Mission Control dashboard endpoints and operator-friendly read models backed by SQLite and vector-enriched summaries.
- **Google Ads Memory RAG**: Stores and retrieves account rules, business context, past explanations, optimization outcomes, and operator preferences for grounded recommendations and conversations.