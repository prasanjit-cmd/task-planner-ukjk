---
name: google-ads-action-planner
version: 1.0.0
description: "Create safe, rationale-rich optimization proposals from Google recommendations, heuristics, and account memory."
allowed-tools: [Bash]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ACTION_REQUIRE_APPROVAL_DEFAULT, GOOGLE_ACTION_MAX_MUTATE_OPS]
---
# Google Ads Action Planner

## What This Skill Does

This skill converts data and recommendations into proposed actions a human can actually review. It takes in Google-native recommendations, internally detected issues, account rules, business guardrails, and prior outcome history, then produces structured proposals with rationale, confidence, and explicit approval status.

This is where the agent becomes operationally useful. It stops being a reporter and starts being a decision-support system.

## Proposal Types

Examples include:
- pause campaign
- enable campaign
- increase or decrease budget
- adjust target CPA or target ROAS
- adjust keyword or ad group bids
- add negative keywords
- change status on paused/eligible entities
- defer action because guardrails conflict

## Inputs

- normalized recommendations
- anomaly alerts
- structure and metrics tables
- account rules
- optimization history and prior outcomes
- remembered user preferences and approval patterns

## Planning Logic

For each candidate action:
1. confirm the target entity still exists
2. gather recent performance context
3. check account rules and protected-entity policies
4. inspect prior similar actions and their outcomes
5. estimate impact and risk
6. determine whether approval is required
7. build the exact proposed payload

## Proposal Fields

Each proposal should include:
- proposal type
- title
- rationale
- proposed payload JSON
- confidence score
- related recommendation id if any
- account and entity references
- requires_approval flag
- approval status

## Guardrails

Never generate an executable proposal that:
- targets a protected entity without calling that out
- exceeds mutate batch safety caps
- lacks a reversible understanding of before-state fields
- contradicts an account rule such as brand budget protection
- relies on stale structure or stale metrics without warning

## Confidence Guidance

Base confidence on:
- consistency of the performance pattern
- presence of corroborating Google recommendations
- similarity to previously successful changes
- strength of account-rule alignment
- data freshness

Confidence should not be inflated to sound decisive. Use it to help prioritization.

## Human Review Style

Proposal summaries should be readable by a paid media operator in seconds. Format each one around:
- action
- target
- why now
- expected upside
- key risk
- whether approval is required

## Steps

1. Load pending recommendations and unresolved alerts.
2. Retrieve applicable account rules and memory context.
3. Generate candidate actions from Google suggestions and local heuristics.
4. Score each candidate for confidence, urgency, and risk.
5. Filter out blocked or noncompliant actions.
6. Serialize the proposal payload and rationale.
7. Write proposals to the queue for review.
8. Link proposals to recommendations where relevant.

## Outcome Learning

Before finalizing a proposal, consult optimization history:
- Which recommendation types are often approved?
- Which actions usually underperform in this account?
- Are there repeated rejections for similar changes?
- Has the user shown a conservative or aggressive preference?

Use this history to personalize prioritization and messaging, not to bypass guardrails.

## Done Criteria

This skill is complete when the system has a queue of clear, reviewable, guardrail-compliant action proposals that operators can approve, reject, or defer with high confidence in what will happen next.