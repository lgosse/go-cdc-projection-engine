---
type: Architecture Decision Record
title: "ADR-0003: Durable state ownership"
description: Kafka owns consumer cursors and engine-owned MongoDB owns control-plane and DLQ state.
tags: [architecture, adr, kafka, mongodb, durability, control-plane]
status: accepted
decision_id: ADR-0003
accepted_on: 2026-08-31
owner: TBD
conditions:
  - DLQ custody is part of the engine-owned MongoDB metadata store.
---

# ADR-0003: Durable state ownership

## Context

Redis has been explicitly made non-authoritative. The engine still needs durable
ownership for Kafka progress, migration coordination, operator actions, and
dead-lettered records. The system draft already uses MongoDB for the
`cdc_projection_dlq` collection, while Kafka provides consumer-group offset
storage.

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
collections. Kafka offset commits are at-least-once progress markers and are not
an atomic transaction with Elasticsearch.

## Condition

DLQ custody—including envelopes, terminal dispositions, replay status, and
operator evidence—is stored in the same engine-owned MongoDB metadata store.

## Boundaries

- Metadata schema, indexes, retention, and deployment placement remain open.
- Exact retry, terminal-error, and offset-commit rules remain open.
- The projector does not write domain-service MongoDB collections.

## Alternatives considered

1. Use a dedicated control-plane database. This isolates projector metadata more
   strongly but adds another platform dependency and compatibility surface.
2. Reconstruct control state from Kafka, MongoDB, Elasticsearch, and deployment
   state. This minimizes durable stores but risks losing in-progress migration
   and terminal failure information.

## Consequences

- MongoDB metadata availability is required for migration operations and DLQ
  custody.
- Crash recovery may replay records, so Elasticsearch operations must be
  idempotent.
- Metadata retention, backup, access control, and capacity are explicit
  operational responsibilities.
- Redis and Elasticsearch remain derived state and must be rebuildable.

## Validation

- Document the state inventory, ownership, retention, and recovery procedure.
- Test crash and rebalance behavior without unsafe offset advancement.
- Test Redis loss without losing migration or DLQ state.
- Restart and fence migrations using only durable metadata.
- Demonstrate DLQ replay and audit history from the same metadata store.

## Review trigger

Revisit if the metadata store cannot meet availability or retention objectives,
Kafka offset durability is insufficient, or a new state type cannot be safely
owned by either authority.

## Related concepts

- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [Redis authority boundary](../01-system-context/redis-authority-boundary.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
