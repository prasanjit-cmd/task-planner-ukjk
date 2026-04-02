---
name: google-ads-outcome-evaluator
version: 1.0.0
description: "Evaluate post-change outcomes so optimizations can be learned from rather than merely executed."
allowed-tools: [Bash]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_REPORTING_TIMEZONE_OVERRIDE]
---
# Google Ads Outcome Evaluator

## What This Skill Does

This skill measures what happened after an executed action. It compares pre-change and post-change windows, records observed movement in spend, conversions, CPA, and ROAS, and converts raw outcomes into reusable learning for future planning and explanation.

Without this skill, the agent would know what it changed but not whether the change helped.

## Goals

- quantify outcome windows after execution
- distinguish expected short-term volatility from likely sustained effect
- provide evidence for future proposal ranking
- improve conversational answers about what past optimizations did

## Inputs

- execution records
- linked proposals
- performance metrics tables
- account rules and context
- relevant measurement window settings

## Window Logic

Common windows include 3, 7, and 14 days depending on volume and change type. Small bid or budget changes may need shorter initial checks, while broader strategy changes may need a longer evaluation window.

The evaluator should support multiple windows rather than pretending one view is enough.

## Measures

Record at least:
- spend_before / spend_after
- conversions_before / conversions_after
- cpa_before / cpa_after
- roas_before / roas_after
- measured_at
- measurement_window_days

Where attribution delay is significant, outcome summaries should include cautionary language.

## Interpretation Rules

Avoid simplistic claims. A post-change outcome should consider:
- whether seasonality or external context explains movement
- whether overlapping changes happened during the same window
- whether data volume is sufficient for confidence
- whether the measured outcome is directional, mixed, or inconclusive

## Steps

1. Find executed proposals whose measurement windows are due.
2. Retrieve pre-change baseline metrics.
3. Retrieve post-change metrics for the configured windows.
4. Calculate deltas and derived efficiency metrics.
5. Classify the outcome as positive, negative, mixed, or inconclusive.
6. Store outcome records.
7. Write a narrative summary for memory and dashboard use.

## Learning Loop

Outcome data should feed back into planning. For similar future proposals, the system can ask:
- have budget increases on this campaign historically improved volume efficiently?
- do target CPA relaxations usually degrade ROAS too much?
- does this user approve actions that later prove neutral?

Use the history as evidence, not as a substitute for current data.

## Done Criteria

This skill is complete when every eligible executed action has measurable before/after records and a structured interpretation that helps operators judge whether the optimization was worthwhile.