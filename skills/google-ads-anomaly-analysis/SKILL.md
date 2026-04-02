---
name: google-ads-anomaly-analysis
version: 1.0.0
description: "Detect meaningful performance anomalies, pacing risk, and account health issues from synced Google Ads metrics."
allowed-tools: [Bash]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ALERT_CPA_SPIKE_THRESHOLD_PCT, GOOGLE_ALERT_SPEND_SPIKE_THRESHOLD_PCT, GOOGLE_REPORTING_TIMEZONE_OVERRIDE]
---
# Google Ads Anomaly Analysis

## What This Skill Does

This skill turns raw performance facts into operational attention. It compares recent metrics to trailing baselines, configured targets, pacing expectations, and remembered business context to detect issues worth surfacing as alerts.

Not every fluctuation matters. This skill’s job is to separate normal volatility from actionable deterioration or surprising improvement.

## Primary Alert Families

- spend spike
- spend drop
- CPA spike
- ROAS deterioration
- CTR drop
- conversion drop
- pacing risk against budget or period targets
- optimization score deterioration when materially relevant
- stale data / sync gap conditions

## Inputs

- daily metrics tables
- account structure tables
- account rules and thresholds
- remembered business context such as seasonality or protected campaigns
- configured threshold percentages

## Alert Design Principles

1. Alerts must be explainable.
2. Alerts must name the metric, current value, baseline, threshold, and affected scope.
3. Alerts should be de-duplicated so the dashboard is not flooded.
4. Alert severity should reflect business impact, not just statistical movement.

## Baseline Logic

Use practical trailing baselines such as:
- prior 7 complete days
- prior 14 days for steadier comparisons
- same weekday comparisons where appropriate
- account rules or target CPA/ROAS when explicitly defined

Where seasonality is recorded in memory or rules, soften or contextualize alert confidence.

## Steps

1. Load fresh metrics and account rules.
2. Compute recent-period snapshots for each account and campaign.
3. Compute trailing baselines and compare deltas.
4. Evaluate configured thresholds for spend and CPA anomalies.
5. Generate structured alerts with severity and rationale.
6. Resolve or downgrade prior alerts when conditions normalize.
7. Write alert rows and summary rollups.
8. Generate a concise health summary for dashboard and chat use.

## Severity Guidance

- `critical`: high-spend or severe efficiency deterioration requiring prompt attention
- `high`: clearly material underperformance or pacing risk
- `medium`: meaningful change worth review soon
- `low`: informational or emerging pattern

## Rationale Style

Each alert description should answer:
- what changed
- compared to what
- why it matters
- likely next investigation or action

Example:
"CPA rose 38% yesterday versus the trailing 7-day baseline while conversions fell 19% and spend increased 11%. This is concentrated in two non-brand search campaigns and may reflect query mix or bid pressure."

## False Positive Control

Do not create high-severity alerts when:
- volume is too low for the metric to be stable
- the campaign is clearly experimental and flagged as such
- a recent approved action plausibly explains the movement and it is within an expected measurement window

Instead, downgrade severity or attach context.

## Done Criteria

This skill is complete when the system has current, actionable alerts and health summaries that help operators understand where attention is needed now, without drowning them in routine noise.