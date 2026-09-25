---
type: Architecture Review Topic
title: Disaster recovery
description: Defines restoration paths for lost caches, indices, checkpoints, and Kafka history.
tags: [lifecycle, recovery, resilience]
status: accepted
decision_id: ADR-0019
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Engine-owned MongoDB metadata, including fences, DLQ custody, migration state, checkpoints, and operator history, is backed up with tested restore/PITR procedures.
  - Redis and Elasticsearch are treated as derived and rebuilt from MongoDB plus retained CDC; Elasticsearch snapshots are optional acceleration only.
  - Recovery fails closed when authority state, deletion evidence, or Kafka history is missing rather than guessing.
  - Raw replay older than 30 days is unsupported; recovery outside that fence-retention window rebuilds current source state before writes resume (ADR-0050).
  - Recovery exercises and per-state RPO/RTO objectives remain required follow-up evidence.
---

# Disaster recovery

## Decision

Use a failure-specific recovery matrix and fail closed whenever authoritative
state or deletion evidence is uncertain:

| Failure | Recovery direction |
| --- | --- |
| Redis loss | Rebuild a new cache generation from MongoDB and retained changes; pause only relations requiring unavailable lookups. |
| Elasticsearch loss or corruption | Provision a fresh versioned target, bootstrap it, replay live overlap, validate, and cut over; snapshots may accelerate this but are not authoritative. |
| Engine metadata MongoDB loss | Restore from backup or point-in-time recovery before resuming; never infer fences, DLQ custody, migration phases, or checkpoints from derived stores. |
| Consumer restart | Restore metadata and resume committed Kafka offsets; replay idempotently and never commit from in-memory state alone. |
| Kafka history expiry or recovery outside 30 days | Keep writes paused and rebuild from current authoritative MongoDB state using a new source boundary; do not resume raw replay older than 30 days. Missing deletion fences or terminal DLQ evidence becomes an auditable incident, not silently accepted recovery. |
| Domain MongoDB unavailable | Keep existing search reads where possible, but pause affected progress and repairs; do not fabricate source state. |
| Full deployment or region loss | Restore metadata first, verify authority and fencing epochs, then resume consumers and rebuild derived state as needed. |

Back up engine-owned MongoDB with a recovery point suitable for deletion fences,
DLQ custody, migration state, checkpoints, and audit history. Kafka retention
must cover the longest expected outage and replay window where replay is
required. Raw replay is supported only within 30 days; after that, rebuild from
current source state and validate before writes resume (ADR-0050). Redis backups
are optional and never replace generation rebuilds.

Recovery is complete only after restored offsets, fences, DLQ records, migration
state, cache generations, and projection convergence have been verified. Define
separate RPO/RTO objectives for metadata and DLQ custody, live processing, search
freshness, and derived-state rebuilds; do not collapse them into one global
number.

## Pros

- Tests the claim that the engine and its projections are rebuildable.
- Exposes hidden durable state before implementation.
- Aligns backup cost with actual authority.

## Cons and risks

- Full rebuild time may exceed acceptable recovery objectives.
- Cross-service source availability can dominate restoration time.
- Recovery procedures require regular exercises to stay credible.
- Metadata backup and point-in-time recovery become mandatory operational
  responsibilities.

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

## Review trigger

Revisit if source or Kafka retention changes, metadata restore cannot meet the
required objectives, regional failover introduces split writers, or rebuilds
impact search freshness or live-ingestion capacity.

## Related concepts

- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Bootstrap consistency](bootstrap-consistency.md)
- [Reconciliation and repair](reconciliation-and-repair.md)
- [Blue-green migration](blue-green-migration.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)

## Follow-up questions

- What are recovery time and recovery point objectives per mode?
- How long must Kafka retention exceed the maximum outage and recovery window?
- What backup/PITR evidence makes metadata recovery valid?
- Who authorizes recovery when deletion fences or DLQ history cannot be
  restored?
- What regional failover procedure prevents two projector deployments from
  writing concurrently?
