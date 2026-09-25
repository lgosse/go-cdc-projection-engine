---
type: Architecture Decision Record
title: "ADR-0046: Source fallback eligibility"
description: Defines which v1 relation lookups may use bounded source-of-truth fallback and the safe behavior when limits are absent.
tags: [architecture, adr, cache, fallback, mongodb]
status: accepted
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - V1 fallback is limited to explicitly allow-listed direct reference-value lookups backed by a unique source index.
  - Reverse-index misses use rebuild, repair, or bounded pending rather than synchronous source fallback.
  - Fallback remains disabled unless finite per-relation limits and an aggregate source-wide ceiling are configured.
  - Numeric budgets are resolved with source-capacity evidence before any fallback-enabled relation is released.
---

# ADR-0046: Source fallback eligibility

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

ADR-0043 establishes default-deny, relation-level source fallback. ADR-0014
distinguishes reference-value caches from reverse indexes, while ADR-0040 places
fallback policy and capacity limits in the versioned manifest contract. Follow-up
Q-120 asked which relation types may read through and what budgets apply. The
eligibility boundary can be decided now; safe numeric limits require capacity
evidence for a representative source and remain tracked by Q-122.

## Decision

For v1, source fallback may be enabled only for an explicitly allow-listed
direct reference-value lookup that uses a unique source index and can return at
most one record. The manifest declares the relation-level timeout, request-rate,
and in-flight concurrency limits. Runtime configuration enforces an aggregate
ceiling across all fallback-enabled relations for each source service. Concurrent
lookups for the same key are deduplicated.

After a read, the engine validates the result against the applicable source
revision or fence before repopulating Redis or using the value in a projection.
If the source read fails, the relation limit is exhausted, or the source-wide
ceiling is reached, the engine does not publish an incomplete projection; it
uses the relation's bounded pending, repair, or rebuild path.

V1 does not synchronously query the source to recover a missing reverse index.
Such misses use asynchronous rebuild, repair, or bounded pending. A relation
with missing, non-finite, or unreviewed limits cannot perform fallback. Numeric
per-relation and source-wide values must be supported by source-owner capacity
limits and representative-projection evidence before a fallback-enabled
manifest is released; Q-122 tracks that contract-package gate.

## Alternatives considered

1. **Allow no source fallback in v1.** This provides the strongest source
   isolation and avoids the need for runtime source budgets, but a cold or
   evicted reference cache can leave valid events pending for rebuild or repair.
2. **Allow selected reverse-index fallbacks as well.** An explicitly indexed
   query with a result cap could restore root context sooner, but adds fan-out,
   result-size, and source-load controls. No first-release evidence currently
   demonstrates those controls are safe.
3. **Allow fallback for every relation miss.** This improves apparent
   recoverability but makes source load and query cardinality difficult to bound
   across projections, so it is rejected.

## Consequences

- A cold direct reference lookup can recover without waiting for a full cache
  rebuild when its relation has approved finite limits.
- Reverse-index loss may delay affected events while the index is rebuilt or
  repaired.
- Every enabled source relation needs measured per-relation limits, and the
  deployment needs an aggregate source ceiling.
- Fallback is unavailable until those values are recorded and validated.

## Validation

- An allow-listed unique-key reference miss performs at most one source read,
  deduplicates concurrent same-key requests, validates freshness, and repopulates
  Redis safely.
- A missing reverse-index key performs no synchronous source query and reaches
  the configured rebuild, repair, or pending path.
- Missing limits, exhausted limits, and source outages never produce an
  incomplete projection or bypass the aggregate source ceiling.
- Measured fallback rates, latency, concurrency, and source load remain within
  the approved source-owner budget.

## Review triggers

Revisit if source capacity evidence supports bounded reverse-index reads, direct
reference fallback threatens source services, relation manifests cannot express
the required limits, or pending/rebuild delays violate freshness objectives.

## Related concepts

- [Source draft conflicts](../source-conflicts.md)
- [Objectives and boundaries](../01-system-context/objectives-and-boundaries.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
