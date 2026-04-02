---
name: google-ads-metrics-sync
version: 1.0.0
description: "Collect daily Google Ads performance facts across core reporting grains with quota-safe scope and retention controls."
allowed-tools: [WebFetch]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_SYNC_HIGH_PRIORITY_ACCOUNT_IDS, GOOGLE_REPORTING_TIMEZONE_OVERRIDE, GOOGLE_ADS_DEFAULT_CURRENCY]
---
# Google Ads Metrics Sync

## What This Skill Does

This skill pulls performance data from Google Ads and writes it into durable daily fact tables. It powers nearly everything users care about: KPI cards, trend charts, campaign rankings, anomaly detection, recommendation context, and conversational analysis.

The goal is not to mirror every possible Google Ads report. The goal is to maintain a practical, explainable, and quota-safe warehouse of operational metrics.

## Required Grains

At minimum, sync:
- account daily metrics
- campaign daily metrics
- ad group daily metrics
- keyword daily metrics
- search term daily metrics for scoped or high-priority accounts

## Core Metrics

Capture as available:
- impressions
- clicks
- cost_micros
- conversions
- conversions_value
- ctr
- average_cpc_micros
- average_cpm_micros where relevant
- cost_per_conversion
- roas
- search impression share fields where available
- optimization_score where available

## Freshness Strategy

Use two patterns:
1. previous-day complete sync for durable reporting
2. intra-day top-line pacing sync for current-day spend and KPI monitoring

This balances completeness with quota safety.

## Search Term and Keyword Growth Control

Keyword and search term data can explode in volume. Apply explicit scope controls:
- prioritize high-priority account ids for deeper syncs
- allow top-N or recent-window filters when needed
- retain enough history for analysis without letting SQLite grow unbounded

## Time Zone Rules

Use the account time zone for reporting whenever possible. If a reporting timezone override is configured, apply it consistently in dashboard summaries and explain that normalization in metadata.

## Steps

1. Determine sync mode: previous-day full or intra-day pacing.
2. For each selected account, start a `metrics_sync` job.
3. Pull account-level daily metrics.
4. Pull campaign-level daily metrics.
5. Pull ad-group-level daily metrics.
6. Pull keyword-level daily metrics where enabled.
7. Pull search-term daily metrics for high-priority accounts or scoped windows.
8. Upsert facts using stable uniqueness constraints.
9. Compute derived fields such as ROAS if needed when not directly returned.
10. Record API operation counts and job status.

## Derived Metric Rules

- ROAS = conversions_value / cost when cost > 0
- CPA = cost / conversions when conversions > 0
- Preserve raw micros and floats so display and analysis layers can choose precision
- Avoid silently substituting zeros for missing share metrics; use null where the metric is not available

## Quality Checks

Before success:
- compare row counts to prior sync ranges
- flag sharp drops in returned entities that suggest query issues
- verify date alignment
- ensure cost and clicks are nonnegative
- ensure primary account-level totals are plausible relative to campaign rollups

## Quota Protection

If operation budget is low:
- prioritize account and campaign sync
- defer keyword and search-term sync
- mark partial freshness status explicitly
- log the skipped lower-priority workloads

## Failure Handling

Retry retryable transport or 5xx failures with exponential backoff and jitter. Do not loop indefinitely on semantic query errors. If only one grain fails, persist the others and raise a targeted warning instead of dropping the whole job silently.

## Done Criteria

This skill is complete when recent performance data is stored at the required grains, freshness metadata is updated, and downstream analysis can answer questions like spend spikes, CPA shifts, underperforming campaigns, and pacing status without making live API calls for every user question.