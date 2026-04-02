---
name: google-ads-mutation-executor
version: 1.0.0
description: "Safely execute approved Google Ads mutations with bounded batches, audit logging, and before/after traceability."
allowed-tools: [WebFetch]
user-invocable: true
metadata:
  openclaw:
    requires:
      env: [GOOGLE_ACTION_REQUIRE_APPROVAL_DEFAULT, GOOGLE_ACTION_MAX_MUTATE_OPS, GOOGLE_ADS_LOGIN_CUSTOMER_ID]
---
# Google Ads Mutation Executor

## What This Skill Does

This skill is the only skill allowed to perform Google Ads mutations. It takes approved action proposals, validates them against current state and account rules, captures before-state snapshots, executes bounded mutate requests, and records the result in a way that operators can audit later.

This is the highest-risk skill in the system. Its behavior must be strict, explicit, and conservative.

## Preconditions

A proposal may be executed only if:
- the target entities still exist
- required approval has been granted
- the proposal is not blocked by account rules
- the payload size and operation count fit safety caps
- before-state can be captured for the fields being changed
- auth context is valid for the target customer account

## Execution Responsibilities

- re-validate the proposal immediately before mutation
- materialize exact mutate payloads
- split large changes into bounded requests
- attach required headers including login-customer-id where needed
- capture request ids and response payloads
- classify full success, partial success, and failure cleanly
- write execution and audit records

## Safety Limits

Google permits large mutate requests, but this system should default lower. Respect `GOOGLE_ACTION_MAX_MUTATE_OPS` and prefer ranges like 500 to 2000 operations per request for better diagnostics and lower blast radius.

## Before-State Snapshots

For each mutable field changed, capture the current value before execution. This supports auditability and better post-change explanation. Store snapshots in request or response payload records or related JSON fields.

## Partial Failure Handling

Treat partial failure as distinct from success. Record:
- which operations succeeded
- which failed
- error codes and messages
- whether the proposal is fully applied, partially applied, or needs manual intervention

Never collapse partial failure into a generic success state.

## Steps

1. Load the approved proposal.
2. Validate approval status and account rules.
3. Refresh current entity state using structure references if needed.
4. Capture before-state snapshot of mutable fields.
5. Build exact Google Ads mutate operations.
6. Split operations into bounded requests.
7. Execute mutate calls with request-id logging.
8. Persist execution records with payloads and errors.
9. Update proposal and related recommendation states.
10. Emit activity-feed events and failure alerts when necessary.

## Error Policy

Retry only retryable transport and temporary service failures when safe. Do not blindly retry semantic mutation errors such as invalid field combinations, removed entities, or rule conflicts.

## Operator Communication

For every execution result, summarize:
- what was attempted
- against which account/campaign/ad group/keyword
- whether it succeeded, partially succeeded, or failed
- the request id
- any follow-up action needed

## Done Criteria

This skill is complete when an approved proposal has either been executed with a fully traceable audit record or stopped with a clear reason, without any ambiguity about what did or did not change in Google Ads.