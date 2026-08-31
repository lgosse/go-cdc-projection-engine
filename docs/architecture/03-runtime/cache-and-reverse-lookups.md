---
type: Architecture Review Topic
title: Cache and reverse lookups
description: Defines cache ownership, completeness, invalidation, and miss behavior.
tags: [runtime, redis, cache, relationships]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: in-review
---

# Cache and reverse lookups

## Decision to stamp

Define how reference values and root-resolution indexes enter Redis, stay fresh,
recover after loss, and behave on misses.

## Draft proposal

Treat cached reference values as refreshable and reverse relations as required
operational indexes with explicit builders. Populate both from source events and
bootstrap jobs. In stream mode, do not query foreign MongoDB databases on miss
by default. A relation may explicitly permit controlled source-of-truth
read-through with per-key deduplication, rate limits, and timeouts; otherwise
retry boundedly, defer the event, or route it to a repair path according to the
relation's declared miss policy.

## Pros

- Preserves source-service isolation at runtime.
- Makes cache completeness and recovery observable.
- Relation-specific miss policy avoids one unsafe global fallback.

## Cons and risks

- TTL eviction can break required root resolution.
- Deferred events need durable ordering and retry ownership.
- Cache warm-up can delay readiness or cause a miss storm.
- Read-through fallback can add source-service load and must be protected.

## Questions to stamp

- Which keys may expire and which must be retained?
- How is a full cache rebuild coordinated with live writes?
- What staleness is acceptable for reference values?
- Which relation types may use controlled source-of-truth read-through?

## Related concepts

- [Stream pipeline](stream-pipeline.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Backpressure, retry, and DLQ](backpressure-retry-dlq.md)
