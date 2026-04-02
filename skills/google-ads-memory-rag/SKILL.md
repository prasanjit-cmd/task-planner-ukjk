---
name: google-ads-memory-rag
version: 1.0.0
description: "Maintain retrieval-augmented memory for Google Ads account rules, business context, prior analyses, and optimization history."
allowed-tools: [WebFetch]
user-invocable: false
metadata:
  openclaw:
    requires:
      env: [VECTOR_DB_URL, VECTOR_DB_API_KEY]
---
# Google Ads Memory RAG

## What This Skill Does

This skill is the long-term contextual memory layer for the Google Ads agent. It stores durable business constraints, learned operator preferences, prior performance explanations, optimization histories, and important conversation snippets so the agent can reason with continuity instead of treating each interaction as stateless.

## Why It Matters

Paid media operations are full of local context that does not live in the API:
- branded campaigns that must not be budget-cut
- approval rules by client or account
- preferred reporting windows
- whether the team is currently in an aggressive growth phase
- how the operator names certain campaigns in conversation
- repeated explanations for known seasonality

The vector store preserves this context in a retrievable form.

## Memory Categories

Store embeddings in purpose-specific collections for:
- durable account rules and constraints
- performance analyses and explanations
- optimization history and outcome narratives
- conversation-derived preferences and aliases
- normalized API failure lessons and recovery notes

## What To Remember

Remember content that changes decisions or improves future explanations. Examples:
- target CPA/ROAS thresholds
- protected campaigns and budget rules
- preferred tone and summary style for reports
- campaigns marked experimental
- recurring seasonal events
- recommendation types the user usually rejects

Do not embed raw secrets or tokens.

## Retrieval Rules

When planning actions or answering questions, retrieve the most relevant account-specific memory first. Favor structured rules when they exist; use vector memory to enrich, explain, and disambiguate.

Examples:
- before proposing a budget cut, retrieve protected budget rules
- before explaining a spend spike, retrieve prior seasonality notes
- before ranking recommendations, retrieve past approval patterns and outcomes

## Write Triggers

Create or update embeddings when:
- account rules are created or updated
- analysis jobs produce meaningful summaries
- proposals are approved, rejected, or evaluated
- a conversation contains a clear memory-worthy instruction or preference
- a novel API failure lesson is normalized

## Steps

1. Accept structured or narrative memory candidates.
2. Classify them into the proper collection.
3. Normalize metadata such as account id, campaign id, topic, and timestamp.
4. Generate embeddings and write them to the vector store.
5. On retrieval, search the most relevant collection set first.
6. Return concise grounded context to planning, analysis, and chat layers.

## Quality Rules

- Prefer short, high-signal chunks over bloated transcripts.
- Include metadata for account and topic scoping.
- Avoid duplicate embeddings for the same resolved event.
- Re-embed updated rules when the rule changes materially.

## Done Criteria

This skill is complete when the agent can reliably retrieve business constraints, prior reasoning, and learned preferences that improve recommendations and explanations without relying on live API data alone.