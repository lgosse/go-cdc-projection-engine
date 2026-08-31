---
type: Architecture Decision Record
title: "ADR-0002: Redis authority boundary"
description: Redis is non-authoritative for projector progress and control-plane state.
tags: [architecture, adr, redis, durability, recovery]
status: accepted
decision_id: ADR-0002
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0002: Redis authority boundary

## Context

The system draft describes Redis as a non-persistent operational lookup cache,
but also uses `cache:active_write_indices:<index_name>` to control dual-write
destinations. Offset progress, migration state, and DLQ custody need recovery
semantics that cannot depend on an evictable or stale cache key.

## Decision

Redis is never authoritative for projector progress or control-plane state. It
may hold refreshable lookup data, reverse indexes, and derived routing data.
Kafka offset progress, migration state, DLQ custody, and any state required for
safe recovery must be stored durably elsewhere or be deterministically
reconstructible from durable sources.

## Boundaries

- The durable owner for checkpoints, migration state, and control-plane metadata
  is intentionally not selected by this ADR.
- Redis persistence, replication, and backup policy cannot make cache contents
  authoritative by default.
- Cache-miss, reverse-index retention, and routing-key refresh policies remain
  separate decisions.

## Alternatives considered

1. Make Redis authoritative using persistence, replication, and backups. This is
   simple and fast, but makes cache infrastructure a correctness dependency.
2. Reconstruct all operational state from Kafka, MongoDB, Elasticsearch, and
   deployment state without a durable control-plane store. This minimizes new
   state, but may be unable to represent in-progress migrations and exact
   partition progress safely.

## Consequences

- Redis loss is a cache/rebuild incident, not silent loss of committed progress.
- Active write targets need a durable or reconstructible source of truth.
- Offset, DLQ, and migration recovery procedures must identify evidence outside
  ephemeral cache contents.
- A durable control-plane boundary or deterministic reconstruction process is
  required before production operation.

## Validation

- Demonstrate Redis loss and restart without losing committed progress or
  migration safety.
- Document every projector state item and its durable/reconstructible source.
- Prove a missing or stale Redis key cannot move a migration into an unsafe
  phase.
- Exercise offset, DLQ, and migration recovery from authoritative evidence.

## Review trigger

Revisit if Redis becomes the only practical source for a required state item, if
platform durability guarantees change, or if recovery objectives cannot be met
without authoritative Redis state.

## Related concepts

- [Redis authority boundary](../01-system-context/redis-authority-boundary.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
