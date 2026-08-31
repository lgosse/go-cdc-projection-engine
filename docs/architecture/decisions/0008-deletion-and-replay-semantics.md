---
type: Architecture Decision Record
title: "ADR-0008: Deletion and replay semantics"
description: Deletes are first-class fenced events with durable root and child deletion custody.
tags: [architecture, adr, deletion, replay, tombstones]
status: accepted
decision_id: ADR-0008
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Deletion fences use the accepted identity and source-scoped ordering rules.
  - Fences remain valid for the maximum supported replay, retention, recovery, and reconciliation horizon.
  - Reference deletion policy, soft-delete visibility, and fence cleanup are separate follow-up decisions.
---

# ADR-0008: Deletion and replay semantics

## Context

The source draft proposes short-lived Redis tombstones to prevent deleted data
from being resurrected. Redis is intentionally non-authoritative, and replay,
recovery, or delayed delivery can exceed a fixed five-minute window. Root and
nested-child deletes also require different projection behavior.

## Decision

Treat deletes as first-class, versioned events using manifest-defined identity
and source-scoped freshness fencing.

For a root entity delete, remove the entity from Elasticsearch and persist a
durable deletion fence in engine-owned MongoDB metadata. A later valid create or
update may recreate the projection only when it carries a newer source fence.

For a nested child delete, remove only the matching child from the parent
projection and persist a fence for that child identity. An older child event
must not restore the removed member.

Deletion fences must survive Redis loss and remain valid for at least the
maximum supported replay, Kafka-retention, recovery, and reconciliation horizon.
A fixed five-minute Redis tombstone is not a correctness boundary.

Events without usable identity or ordering metadata follow the accepted
unprocessable-event and defer/repair rules; the engine must not guess from a
timestamp or Kafka offset.

## Conditions and boundaries

- This ADR does not choose between hard-delete and visible soft-delete markers
  in Elasticsearch.
- Reference-delete behavior (`null`, default, removal, or error) is a separate
  contract decision.
- Fence garbage collection requires a later retention and watermark decision.
- Whether a child event may recreate a deleted root is deferred until root and
  child lifecycle semantics are specified together.

## Alternatives considered

1. Keep deletion markers only in Redis with a short TTL. This is simple and
   limits durable storage, but deleted entities can resurrect after expiry and
   correctness depends on a rebuildable cache.
2. Apply ordinary updates after a marker expires with no durable fence. This
   avoids metadata bookkeeping but cannot distinguish a legitimate recreation
   from a stale replay.

## Consequences

- Replay convergence covers creates, updates, and deletes symmetrically.
- MongoDB metadata grows with retained deletion fences and needs safe cleanup.
- Elasticsearch hard deletion reduces immediate diagnostic evidence; soft-delete
  visibility remains open.
- Child-level fencing increases metadata and mutation complexity but prevents
  stale nested events from restoring removed members.
- Bootstrap and reconciliation must honor retained deletion fences.

## Validation

- Replay duplicate and reordered root deletes and verify convergence.
- Apply a late update after a delete and verify that it cannot resurrect the
  projection.
- Apply a newer create after a delete and verify that legitimate recreation is
  possible.
- Delete one nested child and verify that the parent and other children remain.
- Lose Redis during replay and verify that MongoDB fencing still protects
  deletion correctness.
- Verify bootstrap and reconciliation do not resurrect entities covered by a
  retained deletion fence.

## Review trigger

Revisit if the source cannot provide a comparable fence, if retention or replay
horizons change, or if privacy requirements require a different Elasticsearch
deletion or metadata-erasure policy.

## Related concepts

- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
