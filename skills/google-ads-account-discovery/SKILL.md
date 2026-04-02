---
name: google-ads-account-discovery
version: 1.0.0
description: "Discover accessible customer accounts, MCC relationships, and sync eligibility for Google Ads management."
allowed-tools: [WebFetch]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ADS_LOGIN_CUSTOMER_ID, GOOGLE_ADS_CUSTOMER_IDS]
---
# Google Ads Account Discovery

## What This Skill Does

This skill determines which Google Ads customer accounts the agent can actually see and manage. It starts from authenticated Google Ads access, calls accessible customer discovery endpoints, expands manager-account relationships where needed, and produces a normalized list of accounts eligible for sync and action workflows.

This skill is the bridge between raw auth and meaningful account operations. Without it, the agent may have credentials but no reliable map of which accounts exist, which belong under an MCC, and which should be included or excluded.

## Responsibilities

- List accessible customers.
- Resolve manager-account hierarchy when the integration is MCC-based.
- Apply allowlist filtering if specific customer ids were configured.
- Capture account metadata needed for downstream workflows.
- Record whether an account is a manager, leaf account, or both.
- Prepare selected accounts for structure and metrics sync.

## Core Outputs

- discovered account list
- selected account list after policy filters
- manager-to-client relationship mapping
- initial account metadata snapshot
- operator-readable discovery summary

## Selection Rules

If `GOOGLE_ADS_CUSTOMER_IDS` is set:
- only accounts on the allowlist are marked active for sync
- non-listed discovered accounts may still be visible in diagnostics but not synced automatically

If `GOOGLE_ADS_CUSTOMER_IDS` is not set:
- all accessible non-manager customer accounts may be considered eligible unless explicitly disabled later

## Safety Rules

1. Do not assume an accessible customer is safe to mutate.
2. Discovery success does not imply approval to change settings.
3. Always distinguish manager accounts from spend-bearing leaf accounts.
4. Preserve the original customer ids exactly as canonical identifiers.

## Typical Workflow

1. Receive authenticated request context from `google-ads-auth`.
2. Call accessible customer list endpoint.
3. For each accessible customer, retrieve enough metadata to classify account type and relationship.
4. If operating under an MCC, use manager relationship queries to map parent-child links.
5. Apply configured customer-id allowlist if present.
6. Upsert normalized account rows.
7. Emit a discovery report summarizing counts and selection decisions.

## What To Store

For each discovered account, record:
- stable internal id
- Google customer id
- manager customer id if known
- descriptive name
- currency code
- time zone
- status
- is_manager flag
- connected_at timestamp
- last discovery timestamp
- raw metadata JSON if useful for diagnostics

## Failure Modes

- auth works but accessible customer list is empty
- login-customer-id is wrong for manager traversal
- account metadata query partially fails
- some child accounts are inaccessible due to permissions

Partial discovery should still persist what is known and mark unsupported or failed branches clearly.

## Operator-Facing Summary Style

When presenting discovery results, summarize with:
- number of accessible accounts
- number selected for sync
- number skipped by allowlist
- any manager-account relationship ambiguities
- any accounts lacking sufficient permissions

## Steps

1. Use the authenticated Google Ads context.
2. Fetch accessible customer resource names.
3. Normalize resource names into customer ids.
4. Retrieve descriptive metadata for each account.
5. Determine manager versus leaf account status.
6. Apply configured allowlist and policy filters.
7. Upsert account records and relationship fields.
8. Emit a discovery job record with counts and warnings.

## Done Criteria

This skill is complete when the system has a trustworthy list of accounts it can monitor, an explicit selection of which ones to manage, and a stored mapping that allows downstream sync and mutation skills to route requests to the correct customer ids.