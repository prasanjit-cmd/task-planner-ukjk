---
name: google-ads-dashboard-api
version: 1.0.0
description: "Provide read-optimized Mission Control API endpoints for Google Ads operations data and summaries."
allowed-tools: [WebFetch]
user-invocable: false
metadata:
  openclaw:
    requires:
      env: [SQLITE_DATABASE_PATH]
---
# Google Ads Dashboard API

## What This Skill Does

This skill exposes the read layer used by Mission Control and other internal consumers. It transforms normalized tables and some vector-backed narrative context into stable JSON endpoints that power metric cards, charts, tables, side panels, and activity feeds.

It is not responsible for syncing or mutating Google Ads. It is responsible for making the system legible to humans.

## Design Principles

- Responses should be dashboard-ready.
- Heavy joins and aggregations should be predictable and indexed.
- The API should return concise summaries plus enough detail for drilldowns.
- It should surface stale data and failures rather than hiding them.

## Required Endpoint Families

- overview summary and trends
- accounts listing and account detail
- campaigns listing and campaign detail
- recommendations queue
- actions and outcomes
- alerts
- sync jobs and freshness diagnostics
- manual trigger endpoints for sync and execution where authorized

## Response Design

Every response should aim to support a specific UI need. Avoid generic payloads that force the frontend to reinvent business logic. If the page needs KPI cards and trends, return KPI cards and trends.

## Performance Rules

- use read-optimized SQL
- leverage indexes on account/date/status dimensions
- keep date-range filters explicit
- avoid scanning search-term tables for overview use cases unless truly necessary

## Safety Rules

1. Read endpoints may expose operational metadata but should not leak secrets.
2. Mutating endpoints must validate authorization and proposal state.
3. If data is stale, include freshness warnings in the payload.
4. Keep response shapes stable for dashboard components.

## Steps

1. Accept request parameters such as accountId, status, type, and date range.
2. Translate them into indexed SQL queries.
3. Combine related rows into page-specific summaries.
4. Attach freshness and alert metadata.
5. Return stable JSON shaped for Mission Control components.

## Done Criteria

This skill is complete when Mission Control pages can render directly from API responses without needing expensive client-side data assembly and when operators can trust that each page reflects the latest known synced state plus any freshness caveats.