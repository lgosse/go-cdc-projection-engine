---
type: Architecture Decision
title: Durable state ownership
description: Assigns durable projector state to Kafka offsets and an engine-owned MongoDB metadata store.
tags: [context, kafka, mongodb, durability, control-plane, accepted]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0003
conditions:
  - DLQ custody is stored in the same engine-owned MongoDB metadata store.
---

# Durable state ownership

## Decision

Use a split durable ownership model:

| State | Authority |
| --- | --- |
| Kafka consumption cursor | Kafka consumer-group committed offsets |
| Migration phases, leases, target set, and operator actions | Engine-owned MongoDB metadata database/collections |
| DLQ envelopes and replay status | The same engine-owned MongoDB metadata database/collections |
| Lookup data and reverse indexes | Redis, always rebuildable |
| Projected documents | Elasticsearch, always rebuildable |

The engine-owned MongoDB metadata boundary is separate from domain-service
collections. Kafka offset commits remain at-least-once progress markers, not an
atomic transaction with Elasticsearch.

## Condition

DLQ custody—including the envelope, terminal disposition, replay status, and
operator evidence—belongs in the same engine-owned MongoDB metadata store as
migration and control-plane state.

## Boundaries and non-goals

- This decision does not define the metadata schema, indexes, retention, or
  deployment placement of the engine-owned MongoDB store.
- It does not make Kafka offsets atomic with Elasticsearch writes.
- It does not select exact retry, terminal-error, or offset-commit rules.
- Redis and Elasticsearch remain derived state and must be rebuildable.

## Rationale

Kafka already durably stores consumer-group offsets, while the design already
places DLQ records in MongoDB. A separate engine-owned metadata boundary provides
durable, inspectable migration and failure custody without making Redis a
correctness dependency or writing projector state into domain collections.

## Alternative rejected

Use a dedicated control-plane database or reconstruct all control state from
Kafka, MongoDB, Elasticsearch, and deployment state. The former adds another
platform dependency; the latter risks losing in-progress migration and terminal
failure information.

## Consequences

- MongoDB metadata availability is required for migration operations and DLQ
  custody.
- Kafka commits can cause replay after a crash, so Elasticsearch writes must be
  idempotent.
- Metadata retention, backup, access control, and capacity become explicit
  operational responsibilities.
- Domain-service MongoDB collections remain read-only to the projector.

## Validation evidence

- Document the state inventory, ownership, retention, and recovery procedure.
- Test crash and rebalance behavior without unsafe offset advancement.
- Test Redis loss without losing migration or DLQ state.
- Restart and fence migrations using only durable metadata.
- Demonstrate DLQ replay and audit history from the same metadata store.

## Review trigger

Revisit if the metadata store cannot meet availability or retention objectives,
if Kafka offset durability is insufficient for the delivery model, or if the
projector needs a state type that cannot be safely owned by either authority.

## Related concepts

- [Data-store responsibilities](data-store-responsibilities.md)
- [Redis authority boundary](redis-authority-boundary.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
