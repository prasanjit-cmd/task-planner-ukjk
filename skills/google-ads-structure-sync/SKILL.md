---
name: google-ads-structure-sync
version: 1.0.0
description: "Sync Google Ads entity structure into SQLite for current-state reporting, targeting, and mutation safety checks."
allowed-tools: [WebFetch]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ADS_API_VERSION, GOOGLE_ADS_BASE_URL]
---
# Google Ads Structure Sync

## What This Skill Does

This skill pulls stable account structure from Google Ads and stores it as the local operational model. It captures the parts of account state that are necessary for dashboards, recommendations, and safe mutation execution: accounts, budgets, campaigns, ad groups, keywords, and selected metadata such as statuses, channel types, bidding strategies, budget assignments, and resource names.

Think of this as the entity catalog. If metrics tell you how things performed, structure tells you what those things actually are.

## Why It Matters

Without current structure data, the agent cannot:
- render campaign and ad group tables correctly
- validate that a recommendation still targets a live entity
- detect protected campaigns by name or rule
- construct precise mutate payloads
- show current budget and bidding configuration next to performance

## Sync Scope

At minimum, sync:
- google account metadata
- campaign budgets referenced by campaigns
- campaigns
- ad groups
- keyword criteria
- asset groups only when campaign type and reporting plan require them

## Data Strategy

Use upserts keyed by account and Google entity ids. Preserve `resource_name` for precise follow-up queries and mutations. Record `last_seen_at` on every successful sighting so stale entities can be identified later.

Prefer a full daily structure sync rather than trying to infer entity deletion or renames from sparse metrics.

## Safety Rules

1. Never delete local rows blindly when an entity disappears from a single sync; mark stale based on repeated absence or explicit state.
2. Keep raw identifiers stable even if names change.
3. Preserve status and serving status separately where available.
4. Treat structure as source-of-truth input for mutation validation.

## Query Design Notes

Use GAQL to retrieve current-state resources efficiently. Select only needed fields so operation usage stays controlled. Batch by customer account and paginate cleanly.

Typical fields include:
- campaign.id, campaign.name, campaign.status, campaign.advertising_channel_type
- campaign.bidding_strategy_type, campaign.campaign_budget, campaign.start_date, campaign.end_date
- ad_group.id, ad_group.name, ad_group.status, ad_group.type, ad_group.cpc_bid_micros
- ad_group_criterion.criterion_id, keyword.text, keyword.match_type, status, negative flag

## Steps

1. Receive selected accounts from discovery.
2. Start a `structure_sync` job record per account.
3. Query campaigns and associated metadata.
4. Resolve and persist campaign budget details.
5. Query ad groups for active and recently seen campaigns.
6. Query keyword criteria for relevant ad groups.
7. Upsert account, campaign, ad group, and keyword tables.
8. Update `last_seen_at` on every entity returned.
9. Record counts, request totals, and errors in sync job logs.

## Handling Large Accounts

For large accounts:
- page GAQL queries deterministically
- separate campaigns, ad groups, and keywords into staged queries
- cap nonessential fields
- if quota is tight, prioritize campaigns and ad groups before keyword depth

## Output Quality Checks

Before marking success, confirm:
- every synced campaign belongs to a known account
- every synced ad group resolves to a known campaign
- every keyword resolves to a valid ad group when expected
- resource names are populated for mutable entities
- job counts are plausible relative to the previous sync

## Failure Handling

If some entity layers fail but others succeed:
- persist successful layers
- record partial completion status
- surface stale-state warnings in Mission Control
- do not block metrics sync unnecessarily unless the failure makes joins unsafe

## Done Criteria

The skill is complete when the local database reflects the current operational structure of each selected Google Ads account closely enough that dashboards can render current entities and action workflows can validate targets before proposing or executing changes.