---
type: Architecture Decision Record
title: "ADR-0019: Disaster recovery"
description: Recovery uses authority-aware failure procedures, backs up engine metadata, and rebuilds Redis and Elasticsearch from MongoDB plus retained CDC.
tags: [architecture, adr, disaster-recovery, resilience, backup, restore]
status: accepted
decision_id: ADR-0019
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Engine-owned MongoDB metadata, including fences, DLQ custody, migration state, checkpoints, and operator history, is backed up with tested restore/PITR procedures.
  - Redis and Elasticsearch are treated as derived and rebuilt from MongoDB plus retained CDC; Elasticsearch snapshots are optional acceleration only.
  - Recovery fails closed when authority state, deletion evidence, or Kafka history is missing rather than guessing.
  - Recovery exercises and per-state RPO/RTO objectives remain required follow-up evidence.
---

# ADR-0019: Disaster recovery

## Context

Authority is split across Kafka offsets, engine-owned MongoDB metadata, domain
MongoDB source state, Redis-derived lookups, and Elasticsearch projections. A
single restore procedure cannot safely treat these stores as one transaction.
Recovery must preserve deletion fences, DLQ custody, migration ownership, and
offset semantics while allowing derived projections to be rebuilt.

## Decision

Use a failure-specific recovery matrix and fail closed whenever authoritative
state or deletion evidence is uncertain:

| Failure | Recovery direction |
| --- | --- |
| Redis loss | Rebuild a new cache generation from MongoDB and retained changes; pause only relations requiring unavailable lookups. |
| Elasticsearch loss or corruption | Provision a fresh versioned target, bootstrap it, replay live overlap, validate, and cut over; snapshots may accelerate this but are not authoritative. |
| Engine metadata MongoDB loss | Restore from backup or point-in-time recovery before resuming; never infer fences, DLQ custody, migration phases, or checkpoints from derived stores. |
| Consumer restart | Restore metadata and resume committed Kafka offsets; replay idempotently and never commit from in-memory state alone. |
| Kafka history expiry | Start a new source-bounded MongoDB bootstrap. Missing deletion fences or terminal DLQ evidence becomes an auditable incident, not silently accepted recovery. |
| Domain MongoDB unavailable | Keep existing search reads where possible, but pause affected progress and repairs; do not fabricate source state. |
| Full deployment or region loss | Restore metadata first, verify authority and fencing epochs, then resume consumers and rebuild derived state as needed. |

Back up engine-owned MongoDB with a recovery point suitable for deletion fences,
DLQ custody, migration state, checkpoints, and audit history. Kafka retention
must cover the longest expected outage and replay window where replay is
required. Redis backups are optional and never replace generation rebuilds.

Recovery is complete only after restored offsets, fences, DLQ records, migration
state, cache generations, and projection convergence have been verified. Define
separate RPO/RTO objectives for metadata and DLQ custody, live processing, search
freshness, and derived-state rebuilds; do not collapse them into one global
number.

## Alternatives considered

1. Back up every datastore and restore them together. This can shorten some
   outages but cross-service backups are not transactionally consistent, add
   cost, and leave authority ambiguous.
2. Reconstruct all state from Kafka and current source databases. This avoids
   metadata backups but cannot safely recover expired terminal custody,
   deletion fences, or in-progress migration ownership.
3. Continue processing with partial or inferred recovery state. This improves
   apparent availability but risks duplicate migration owners, resurrection
   after deletes, and untraceable DLQ loss.

## Consequences

- Recovery runbooks must identify authority, restore order, fencing checks, and
  operator sign-off for each failure class.
- Derived-state loss is recoverable, but rebuild time and source availability
  determine search freshness and processing RTO.
- Kafka retention and metadata backup retention become explicit capacity and
  compliance requirements.
- Expired history or missing metadata may require a controlled bootstrap and may
  not prove historical delete or DLQ completeness.

## Validation

- Redis and Elasticsearch can be lost and rebuilt without source-data loss.
- Metadata restoration preserves offsets, fences, DLQ custody, checkpoints, and
  migration leases.
- Kafka replay and source-bounded bootstrap converge without resurrection or
  unsafe offset advancement.
- Expired Kafka history produces a controlled, auditable incident rather than
  silent data loss.
- Recovery exercises meet the agreed per-state RPO/RTO objectives.

## Review triggers

Revisit if source or Kafka retention changes, metadata restore cannot meet the
required objectives, regional failover introduces split writers, or rebuilds
impact search freshness or live-ingestion capacity.

## Related concepts

- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
