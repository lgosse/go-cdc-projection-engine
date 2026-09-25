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
conditions:
  - An independent referenced-entity delete recomputes dependent roots without cascading; optional derived fields are removed and explicitly safe missing-value behavior is applied (ADR-0063).
---

# Deletion and replay

## Decision

Treat deletes as first-class, versioned events using the accepted identity and
source-scoped fencing rules.

For a root entity delete, remove the entity from Elasticsearch and persist a
durable deletion fence in engine-owned MongoDB metadata. A later valid create or
update may recreate the projection only when it carries a newer source fence.

For an `owned_child` delete, remove only that member's projected state and
persist a fence for the child identity, whether its projection storage is
nested or separate. An older child event must not restore the removed member.

For an `owned_child` relation, a child event cannot resurrect a root protected
by a retained root deletion fence. If the projection is missing without such a
fence, child-before-parent handling follows the bounded context-resolution and
orphan rules in the [stream contract](../03-runtime/stream-pipeline.md). Only a
valid root create/update newer than the deletion fence can recreate a deleted
root. Deleting a root removes its projection but does not delete child source
records. Snapshot data has no separate lifecycle fence and follows its root.
These lifecycle rules are defined by
[ADR-0059](../decisions/0059-relation-lifecycle-classification.md).

For an independent referenced-entity delete, keep dependent roots and their
unrelated fields. Treat the delete as a dependency change: resolve affected
roots through the reverse index and recompute with the reference known to be
absent. Remove fields derived only from that reference when they are declared
optional, or apply an explicitly declared safe missing-value rule. Before
production, block a manifest whose required outputs depend on an independent
reference without a safe missing-value rule. If a particular recomputation
cannot complete safely, durably defer and expose it as blocked or pending; do
not DLQ the valid delete or report the dependent root as refreshed. This
resolves the copied-field behavior left open by
[ADR-0059](../decisions/0059-relation-lifecycle-classification.md) and
[ADR-0061](../decisions/0061-projection-dependency-invalidation.md) through
[ADR-0063](../decisions/0063-independent-reference-delete-effects.md).

During a rebuild, treat an omitted child as absent only when the relation was
completely enumerated at a recorded source boundary. Omit that child from the
rebuilt document, retain any durable child fence already known, and process
boundary-overlap changes through the normal fenced path. Do not invent a
child-level revision or fence from absence alone. If scan completeness or the
ordering needed to reject stale overlap is unclear, keep the rebuild pending
and block cutover until authoritative source evidence or reconciliation resolves
the ambiguity ([ADR-0051](../decisions/0051-child-rebuild-from-source-snapshots.md)).

Deletion fences are authoritative in engine-owned MongoDB and must remain valid
for at least 30 days after they are durably recorded, including through Redis
loss. Raw replay older than 30 days is unsupported. Recovery outside that
window must rebuild from current authoritative source state and validate the
source boundary before writes resume. This is an accepted risk boundary, not a
proof that an older event cannot arrive; such an event can recreate deleted
projection state after its fence expires ([ADR-0050](../decisions/0050-deletion-fence-retention-policy.md)).
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
- A hidden, non-searchable Elasticsearch tombstone or derived fence may be kept
  to enforce local stale-update suppression; MongoDB remains authoritative.
- An independent reference delete removes optional copied fields or applies an
  explicitly safe missing-value rule; it never deletes the dependent root
  ([ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).
- Fence retention and cleanup follow the 30-day policy and its accepted residual
  risk ([ADR-0050](../decisions/0050-deletion-fence-retention-policy.md)).
- An independent referenced-entity delete never cascades to dependent roots;
  copied fields follow optionality or explicitly safe missing-value behavior
  ([ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).

## Validation

- Replay duplicate and reordered root deletes and verify convergence.
- Apply a late update after a delete and verify that it cannot resurrect the
  projection.
- Apply a newer create after a delete and verify that legitimate recreation is
  possible.
- Delete one owned child and verify that its root and sibling members remain.
- Lose Redis during replay and verify that MongoDB fencing still protects
  deletion correctness.
- Verify bootstrap and reconciliation do not resurrect entities covered by a
  retained deletion fence.
- Verify a complete source-bounded relation snapshot omits absent children,
  while incomplete scans or unresolved ordering keep the rebuild pending and
  prevent cutover.
- Verify a late owned-child event cannot resurrect a root protected by a
  retained deletion fence, while an ordinary child-before-parent event follows
  the accepted bounded context-resolution and orphan rules.
- Delete an independent reference and verify that only reverse-indexed
  dependent roots are recomputed, optional derived fields are removed or safe
  defaults are applied, and root documents are not cascaded away.
- Verify preflight blocks a manifest with required independent-reference output
  and no declared safe missing-value behavior; a valid reference delete is not
  sent to the poison-event DLQ.

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

- **Resolved by [ADR-0050](../decisions/0050-deletion-fence-retention-policy.md):** What proves a deletion fence is safe to discard? The accepted policy retains it for 30 days and explicitly accepts the residual risk after expiry.
- **Resolved by [ADR-0051](../decisions/0051-child-rebuild-from-source-snapshots.md):** How are projection documents rebuilt when a deleted child is absent from a source snapshot? Omit it only after complete relation enumeration at a source boundary; preserve known fences and block cutover if completeness or stale-event ordering is uncertain.
- **Resolved by [ADR-0063](../decisions/0063-independent-reference-delete-effects.md):** When a referenced independent entity is deleted, what happens to its materialized fields on dependent roots? Recompute dependent roots without cascading; remove optional derived fields or apply explicit safe missing-value behavior, and block unsafe manifests before production.
