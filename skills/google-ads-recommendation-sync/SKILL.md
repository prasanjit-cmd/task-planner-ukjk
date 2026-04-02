---
name: google-ads-recommendation-sync
version: 1.0.0
description: "Ingest Google Ads recommendations, normalize them locally, and maintain a reviewable recommendation queue."
allowed-tools: [WebFetch]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ADS_API_VERSION]
---
# Google Ads Recommendation Sync

## What This Skill Does

This skill retrieves Google-generated recommendations from the Google Ads API and converts them into a local, operator-friendly queue. Raw recommendation resources are useful, but they are not enough on their own. This skill standardizes recommendation types, extracts impact estimates, attaches target entities where possible, and preserves the lifecycle state needed for approval and execution workflows.

## Why It Exists

Google recommendations are a valuable input, not an automatic instruction. The agent needs a durable, normalized layer where recommendations can be:
- reviewed across accounts
- filtered by type, status, and target entity
- compared with local guardrails
- translated into action proposals
- tracked historically even when dismissed or superseded

## Inputs

- authenticated Google Ads context
- selected accounts
- current entity structure for target resolution

## Outputs

- normalized recommendation rows
- impact JSON payloads
- details JSON payloads
- local source tagging (`google`)
- state tracking fields such as applied/dismissed/status

## Normalization Rules

For each recommendation:
- preserve the raw Google recommendation id when available
- classify into a stable local type string
- attach account, campaign, and ad group identifiers when resolvable
- store the raw recommendation payload in `details_json`
- extract impact estimates into `impact_json`
- keep timestamps current on each refresh

## State Rules

Recommendation state must distinguish:
- newly fetched and pending review
- approved into a proposal
- rejected or dismissed
- applied through execution
- stale or no longer present upstream

Do not erase old recommendations merely because they disappeared upstream; preserve history and update state.

## Steps

1. Start a recommendation sync job for each selected account.
2. Query recommendation resources from Google Ads.
3. Parse recommendation subtype and target entity references.
4. Resolve target campaign/ad group when possible using structure tables.
5. Extract estimated impact and serialize details.
6. Upsert recommendation rows.
7. Mark outdated recommendation states where appropriate.
8. Emit counts by recommendation type and status.

## Guardrails

1. Do not auto-apply any Google recommendation directly from this skill.
2. Do not assume recommendation quality; pass everything through local planning logic first.
3. Preserve enough raw payload detail to explain the recommendation later.
4. If a recommendation targets a protected entity, that fact should remain available to downstream planners.

## Operator Experience

The resulting queue should support answers like:
- What recommendations are pending right now?
- Which recommendation types get approved most often?
- What estimated impact is sitting unresolved?
- Which recommendations conflict with account rules?

## Failure Handling

If recommendation retrieval fails:
- do not delete prior recommendation state
- mark sync failure explicitly
- retain prior pending queue with stale-data warning
- separate API failure from empty recommendation set

## Done Criteria

This skill is complete when the system has a fresh, reviewable, historically durable recommendation queue that downstream planning can evaluate alongside account rules, prior approvals, and current performance data.