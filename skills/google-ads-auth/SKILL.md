---
name: google-ads-auth
version: 1.0.0
description: "Authenticate and prepare safe, quota-aware Google Ads REST requests using OAuth 2.0 and developer token headers."
allowed-tools: [WebFetch]
user-invocable: false
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ADS_DEVELOPER_TOKEN, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN, GOOGLE_ADS_API_VERSION, GOOGLE_ADS_BASE_URL, AGENT_ENCRYPTION_KEY]
---
# Google Ads Auth

## What This Skill Does

This skill is responsible for making Google Ads API access reliable and predictable. It prepares authenticated REST calls, refreshes access tokens from the refresh token flow, attaches the required Google Ads headers, validates configuration early, and standardizes request metadata for downstream skills.

The output of this skill is not business insight. Its job is to create a trusted authenticated context so every sync, analysis, recommendation, and mutation task begins from a known-good request posture.

## Responsibilities

- Validate presence of required credentials and API settings.
- Exchange refresh token for access token through Google OAuth.
- Build request headers:
  - `Authorization: Bearer <access_token>`
  - `developer-token: <token>`
  - `login-customer-id: <manager_id>` when configured or required
- Normalize base URL and versioning.
- Tag every outgoing request with request context fields used for logging.
- Fail fast on invalid auth configuration.
- Avoid embedding secrets in logs or user-visible output.

## Inputs

- OAuth client id
- OAuth client secret
- OAuth refresh token
- Google Ads developer token
- API version
- base URL override if present
- optional manager customer id for MCC flows

## Outputs

- short-lived access token
- canonical API base URL
- canonical header template
- auth validation result
- scope notes for account-discovery and downstream calls

## When To Use This Skill

Use it at the start of any job that touches Google Ads API, including:
- account discovery
- structure sync
- metrics sync
- recommendation sync
- mutation execution
- post-action validation reads

## Guardrails

1. Never print secrets into conversational replies.
2. Never store refresh tokens unencrypted at rest.
3. If token refresh fails, stop the workflow immediately and create a clear operator-facing error.
4. If the developer token appears invalid or unauthorized, classify it separately from transient network failure.
5. If login-customer-id is configured, ensure it is header-safe and normalized to digits only.

## Error Handling

Classify errors into these buckets:
- `auth_invalid_client`
- `auth_invalid_grant`
- `auth_missing_scope_or_permissions`
- `developer_token_rejected`
- `network_retryable`
- `rate_limited`
- `unknown_auth_failure`

Retry only retryable network and temporary service failures. Do not retry invalid credentials indefinitely.

## Steps

1. Confirm all required environment variables exist.
2. Normalize API version and base URL.
3. Exchange refresh token for a short-lived access token using Google OAuth token endpoint.
4. Validate token exchange success and expiry metadata.
5. Construct the standard Google Ads request header set.
6. Add optional `login-customer-id` when running under MCC flows.
7. Return a reusable auth object to downstream skills.
8. Emit a structured auth-health result for diagnostics and sync-job logging.

## Logging Requirements

Log only safe metadata:
- auth step name
- success/failure
- HTTP status code
- token expiry timestamp if available
- whether login-customer-id was attached
- downstream customer id target when known

Do not log access tokens, refresh tokens, or client secrets.

## Integration Notes

This skill should be implemented against Google OAuth and Google Ads REST directly instead of relying on a heavy SDK. A thin internal adapter is preferred so request construction remains explicit and quota accounting is easy to reason about.

## Done Criteria

This skill is complete for a run when downstream jobs receive a valid access token and a header template that works for Google Ads requests, or when a clearly classified auth failure is returned with enough detail for an operator to fix configuration quickly.