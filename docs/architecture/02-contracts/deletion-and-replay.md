---
type: Architecture Review Topic
title: Deletion and replay
description: Defines root deletion, nested removal, tombstones, and late-event behavior.
tags: [contracts, deletion, replay, tombstones]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0008
---

# Deletion and replay

## Decision

Treat deletes as first-class, versioned events using the accepted identity and
source-scoped fencing rules.

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

## Pros

- Makes replay convergence symmetric for creates, updates, and deletes.
- Prevents delayed events from silently resurrecting data.
- Forces downstream deletion and privacy requirements into the design.

## Cons and risks

- Durable tombstones can grow without bound.
- Hard deletion reduces diagnostic evidence and repair options.
- Retention-safe cleanup needs watermarks or source compaction guarantees.

## Conditions and boundaries

- This decision does not choose between hard-delete and visible soft-delete
  markers in Elasticsearch.
- Reference-delete behavior (`null`, default, removal, or error) is a separate
  contract decision.
- Fence garbage collection requires a later retention/watermark decision.
- Whether a child event may recreate a deleted root is deferred until root and
  child lifecycle semantics are specified together.

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

- [Identity, time, and ordering](identity-time-ordering.md)
- [Relationship model](relationship-model.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)

## Questions to stamp

- What proves a deletion fence is safe to discard?
- How are projection documents rebuilt when a deleted child is absent from a
  source snapshot?
