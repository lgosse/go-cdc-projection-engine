---
type: Architecture Decision
title: Redis authority boundary
description: Establishes Redis as non-authoritative for projector progress and control-plane state.
tags: [context, redis, durability, recovery, accepted]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0002
---

# Redis authority boundary

## Decision

Redis is never authoritative for projector progress or control-plane state. It
may hold refreshable lookup data, reverse indexes, and derived routing data.
Kafka offset progress, migration state, DLQ custody, and any state required for
safe recovery must be stored durably elsewhere or be deterministically
reconstructible from durable sources.

## Boundaries and non-goals

- This decision does not choose the durable owner for checkpoints, migration
  state, or control-plane metadata.
- Redis persistence, replication, and backup policy remain operational topics;
  they cannot turn a Redis cache key into the correctness authority by default.
- Cache-miss behavior, reverse-index retention, and routing-key refresh policy
  remain separate decisions.

## Rationale

The system draft describes Redis as a non-persistent cache while using it for
active write-index routing. Making those keys authoritative would allow eviction,
failover, or stale restoration to affect processing correctness and migration
safety. A non-authoritative boundary makes Redis loss recoverable and forces the
durable state inventory to be explicit.

## Alternative rejected

Use Redis as the authoritative operational store, relying on persistence,
replication, and backups for checkpoints and migration state. This keeps
coordination local and fast, but makes cache infrastructure a correctness and
recovery dependency.

## Consequences

- Redis loss is a cache/rebuild incident, not silent loss of committed progress.
- Active write targets must have a durable or reconstructible source of truth.
- Offset, DLQ, and migration recovery procedures must identify authoritative
  evidence outside ephemeral cache contents.
- A durable control-plane boundary or deterministic reconstruction process is
  required before production operation.

## Validation evidence

- Demonstrate Redis loss and restart without losing committed progress or
  migration safety.
- Document every projector state item and its durable/reconstructible source.
- Prove a missing or stale Redis key cannot move a migration into an unsafe
  phase.
- Exercise offset, DLQ, and migration recovery from authoritative evidence.

## Review trigger

Revisit if Redis becomes the only practical source for a required state item, if
the platform changes its durability guarantees, or if recovery objectives cannot
be met without authoritative Redis state.

## Related concepts

- [Data-store responsibilities](data-store-responsibilities.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
