---
type: Architecture Review Topic
title: Stream pipeline
description: Defines deterministic live-event processing and coalescing boundaries.
tags: [runtime, streaming, batching, transformations]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0013
conditions:
  - Pending child events have bounded in-memory and durable repair lifecycles.
  - Source-of-truth read-through is permitted only by an explicit relation policy with rate, timeout, and deduplication controls.
  - Missing parents receive an explicit orphan disposition rather than indefinite retention.
  - A coalesced mutation completes all contributing offsets only after one definitive outcome.
---

# Stream pipeline

## Decision

Decode and validate events while retaining the original envelope, resolve
affected root IDs, and batch required lookups. Assemble a canonical per-root
context from the event, existing projection/cache state, and any source-of-truth
read explicitly permitted by the relation's miss policy.

When a child arrives before its parent, keep the event pending only within a
bounded retry/age window. Use a durable pending or repair record when the wait
outlives normal in-memory processing. If the parent is still unavailable, apply
an explicit orphan disposition—such as audited discard, DLQ, or an allowed
partial projection—rather than retaining the event indefinitely. A missing
parent event is not by itself proof that the parent entity does not exist.

Treat Redis misses according to the relation: controlled read-through with
per-key deduplication, rate limits, and timeouts may be allowed for infrequent
reference lookups; reverse-index misses may defer to asynchronous rebuild or
repair. Do not query foreign MongoDB services without an explicit policy, and do
not publish derived values while required context is unavailable.

Apply source-scoped fences, coalesce events by projection document within bounded
batch limits, evaluate Bloblang once against the canonical context, and emit one
idempotent mutation per target document and active physical index. Record the
outcome against every contributing partition offset only after the combined
mutation has a definitive result.

Determine affected projections from the manifest's materialized dependency graph
at source-entity/relation granularity. A participating entity update triggers
recomputation even when its changed field is not directly projected; see
[ADR-0061](../decisions/0061-projection-dependency-invalidation.md). Cache
refreshes and transport-only metadata do not independently trigger recomputation.

## Pros

- Coalescing reduces write amplification.
- A canonical intermediate form supports shared semantics across modes.
- Per-document mutation boundaries match Elasticsearch atomicity.
- Bounded pending and relation-specific fallbacks distinguish late context from
  genuinely missing source data.

## Cons and risks

- Coalescing records from multiple Kafka partitions complicates commits.
- A partial event stream may not reconstruct the canonical document in memory.
- Large hot documents can dominate a batch and create skew.
- Controlled source reads add latency and can overload a source service if miss
  protection is insufficient.
- Pending and orphan custody add lifecycle, retention, and operator-workflow
  complexity.

## Alternatives considered

1. Immediately DLQ every child or cache miss. This keeps latency bounded but can
   lose valid out-of-order events and treats recoverable cache loss as bad data.
2. Keep missing-context events pending indefinitely. This avoids premature loss
   but creates unbounded state and can block a partition forever.
3. Query every foreign MongoDB source synchronously on every miss. This can
   recover data immediately but removes source isolation and risks cascading
   load during cache outages.

## Consequences

- A child event can converge even when its parent CDC event never arrives, if the
  parent exists in an approved source-of-truth lookup.
- Genuine orphans become explicit, bounded terminal cases rather than hidden
  permanent backlog.
- Cache misses can recover without making unrestricted cross-database queries.
- Pending custody, source-read protection, and coalescing outcome mapping become
  observable operational responsibilities.
- Cross-partition coalescing improves efficiency but a failed combined mutation
  can keep several offsets unresolved.

## Validation

- Child-before-parent converges when the parent exists in MongoDB but has no
  recent CDC event.
- An absent parent reaches its configured orphan disposition within the pending
  horizon.
- Redis eviction can recover through permitted read-through or repair without
  publishing incomplete derived fields.
- Source outages do not cause uncontrolled read storms or unsafe partial
  projections.
- Coalesced mutations map one definitive outcome back to every contributing
  partition offset.
- Deleted parents cannot be recreated by late child events.

## Review trigger

Revisit if pending work routinely exceeds its horizon, source read-through
threatens service capacity, orphan rates indicate a contract problem, or
coalescing cannot preserve reliable offset completion.

## Related concepts

- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Cache and reverse lookups](cache-and-reverse-lookups.md)
- [Elasticsearch writes](elasticsearch-writes.md)
- [Offsets and delivery semantics](offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](backpressure-retry-dlq.md)

## Follow-up questions

- What pending age/retry limit and durable custody trigger apply before an event
  becomes an orphan or repair case?
- Which relation types permit controlled source-of-truth read-through on a cache
  miss, and how are those limits represented in the manifest?
- What are the maximum batch age, size, memory, and hot-key fairness budgets?
- Can coalescing cross topic or partition boundaries safely when one combined
  mutation fails?
