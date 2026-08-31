---
type: Architecture Decision Record
title: "ADR-0013: Stream pipeline context resolution"
description: The stream pipeline uses bounded pending and controlled source fallback to assemble deterministic projection mutations.
tags: [architecture, adr, streaming, coalescing, context, repair]
status: accepted
decision_id: ADR-0013
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Pending child events have bounded in-memory and durable repair lifecycles.
  - Source-of-truth read-through is permitted only by an explicit relation policy with rate, timeout, and deduplication controls.
  - Missing parents receive an explicit orphan disposition rather than indefinite retention.
  - A coalesced mutation completes all contributing offsets only after one definitive outcome.
---

# ADR-0013: Stream pipeline context resolution

## Context

Kafka topics are independently partitioned, so a child event may arrive before
its parent event—or the parent event may never be emitted during the engine's
run. Redis is rebuildable and may miss rarely updated references. The pipeline
must resolve enough context for deterministic Bloblang transformation without
waiting forever or publishing unsafe partial projections.

## Decision

Decode and validate events while retaining the original envelope, resolve
affected root IDs, and batch required lookups. Assemble a canonical per-root
context from the event, existing projection/cache state, and any source-of-truth
read explicitly permitted by the relation's miss policy.

When a child arrives before its parent, keep the event pending only within a
bounded retry/age window. Use a durable pending or repair record when the wait
outlives normal in-memory processing. If the parent is still unavailable, confirm
whether it exists in the source of truth and apply an explicit orphan
disposition—such as audited discard, DLQ, or an allowed partial projection—rather
than retaining the event indefinitely. A missing parent event is not by itself
proof that the parent entity does not exist. A retained deletion fence takes
precedence over a late child event.

Treat Redis misses according to the relation: controlled read-through with
per-key deduplication, rate limits, and timeouts may be allowed for infrequent
reference lookups; reverse-index misses may defer to asynchronous rebuild or
repair. Do not issue unrestricted foreign MongoDB queries, and do not publish
derived values while required context is unavailable.

Apply source-scoped fences, coalesce events by projection document within bounded
batch limits, evaluate Bloblang once against the canonical context, and emit one
idempotent mutation per target document and active physical index. Record the
outcome against every contributing partition offset only after the combined
mutation has a definitive result.

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

## Review triggers

Revisit if pending work routinely exceeds its horizon, source read-through
threatens service capacity, orphan rates indicate a contract problem, or
coalescing cannot preserve reliable offset completion.

## Related concepts

- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
