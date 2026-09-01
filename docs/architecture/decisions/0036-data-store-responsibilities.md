---
type: Architecture Decision Record
title: "ADR-0036: Data-store responsibilities"
description: Domain MongoDB and engine metadata own authoritative state while Kafka, Redis, and Elasticsearch have explicit delivery or derived roles.
tags: [architecture, adr, context, kafka, mongodb, redis, elasticsearch, durability]
status: accepted
decision_id: ADR-0036
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Domain MongoDB remains authoritative for business records; engine-owned MongoDB metadata is authoritative for control, custody, fences, and checkpoints.
  - Kafka owns live delivery and consumer cursor progress within its retention window, not canonical business state.
  - Redis reverse indexes may be required for runtime resolution but remain derived and rebuildable.
  - Elasticsearch remains a versioned rebuildable read model exposed only through verified aliases.
  - Prolonged shared metadata disagreement and exact reverse-index inventories remain follow-up topics.
---

# ADR-0036: Data-store responsibilities

## Context

The engine spans domain-service MongoDB collections, Kafka CDC delivery, an
engine-owned MongoDB metadata store, Redis lookup and reverse-index data, and
versioned Elasticsearch read models. Earlier decisions establish individual
boundaries, but recovery and disagreement handling require one consolidated
authority matrix.

## Decision

Assign authority as follows:

| Concern | Authority | Behavior on loss or disagreement |
|---|---|---|
| Domain business records | Domain-service MongoDB | Canonical source for bootstrap, reconciliation, and repair |
| Live CDC delivery | Kafka topics and consumer-group offsets | Process within retention; offsets own cursor progress |
| Migration, checkpoints, leases, fences, DLQ, and replay custody | Engine-owned MongoDB metadata | Maintenance and safe recovery pause if unavailable |
| Reference lookups | Redis derived cache | Read-through or bounded pending according to relation policy |
| Reverse relationship indexes | Redis derived operational indexes | Required for some runtime resolutions, but rebuildable and never authoritative |
| Search documents | Elasticsearch physical targets | Rebuildable from MongoDB plus retained CDC |
| Search visibility | Elasticsearch aliases | Only a verified active target may be exposed |

Domain-service MongoDB and engine-owned MongoDB metadata remain separate
ownership boundaries. The projector does not write control state into domain
collections.

For disagreement, MongoDB-derived state wins for canonical rebuild and
reconciliation. Kafka events apply only when identity and source revision are
valid; stale events are fenced. Kafka offsets and MongoDB metadata own different
state domains and are not made one transaction. MongoDB wins over Redis and
Elasticsearch; those stores are refreshed, reconciled, or rebuilt. Neither Redis
nor Elasticsearch automatically wins over the other.

When a required source boundary or dependency is unavailable, report `unknown`,
hold or pause the narrowest safe scope, and do not invent a value or trigger
blind repair.

Classify Redis relationship data as optional reference caches or required runtime
reverse indexes. Required indexes, such as `dispute.attendance_id -> task_id`,
remain derived and rebuildable. If unavailable, events wait, defer, or enter
repair custody according to their relation policy; the index never becomes
authoritative merely because runtime resolution depends on it.

## Alternatives considered

1. **Kafka as the complete source of truth.** This fails after retention expiry
   and cannot represent current business state as reliably as MongoDB.
2. **Redis owns routing, fences, or progress.** Eviction or failover could
   silently change correctness.
3. **Elasticsearch is canonical after indexing.** Partial writes, mappings, and
   aliases would make the read model its own authority.
4. **One MongoDB store owns domain and engine state.** This couples projector
   mutations to domain-service ownership and access boundaries.
5. **Every reverse index is optional.** Some relationship events then become
   impossible to resolve safely and the dependency remains hidden.

## Consequences

- A maintained state inventory maps every item to authority, retention, and
  recovery source.
- Redis and Elasticsearch loss are rebuild incidents, not authority changes.
- Required reverse-index policies become manifest and operational concerns.
- MongoDB metadata availability is required for migration, DLQ, fencing, and
  checkpoint safety.
- Canonical comparisons and repairs use MongoDB-derived state and documented
  source boundaries.

## Validation

- Redis and Elasticsearch loss does not lose migration, DLQ, fence, or cursor
  state.
- Kafka/MongoDB disagreement follows canonical and fencing rules.
- Required reverse-index loss produces bounded pending, pause, or repair behavior.
- Rebuilds converge to MongoDB-derived canonical state.
- No projector write crosses into domain-service control ownership.
- Every state item has a documented authority and recovery path.

## Review triggers

Revisit if the metadata store cannot meet availability or retention objectives,
Kafka retention or MongoDB rebuilds cannot support recovery, required reverse
indexes change materially, or prolonged source disagreement threatens
correctness.

## Related concepts

- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [Live ingestion source](../01-system-context/live-ingestion-source.md)
- [Redis authority boundary](../01-system-context/redis-authority-boundary.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
