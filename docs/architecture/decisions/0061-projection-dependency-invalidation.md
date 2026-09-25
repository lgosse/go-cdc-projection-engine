---
type: Architecture Decision Record
title: "ADR-0061: Projection dependency invalidation"
description: Defines which source changes trigger projection recomputation in v1.
tags: [architecture, adr, transformations, relationships, recomputation]
status: accepted
decision_id: ADR-0061
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Recompute at the source-entity/relation level for every accepted change in the declared materialized dependency graph.
  - Resolve affected roots through the lifecycle role and declared relation indexes; never infer an empty dependency set from an incomplete index.
  - Field-level invalidation is not required in v1; configuration changes follow the schema and migration contract.
  - Independent-reference delete effects remain blocked under Q-126.
---

# ADR-0061: Projection dependency invalidation

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

A projection can combine root data, owned-child events, and independently
referenced entities. A change to any participating source can make one or more
root projections stale. The engine needs one explicit rule for discovering
which roots to recompute, including multi-hop relations.

Field-level invalidation could avoid recalculating a projection when an entity
changes only in an unused field. It would also require complete, maintainable
field-dependency declarations or reliable static analysis of every allowed
transformation and every source change description. V1 already has a declared
acyclic relation graph, root-resolution rules, and reverse indexes for mutable
references ([ADR-0057](0057-reference-change-propagation.md)).

## Decision

Use entity/relation-level invalidation in v1. Every accepted create or update
event for a source entity that participates in a manifest's materialized
dependency graph triggers recomputation for the roots that depend on that entity,
even when the event changed a field not directly used in the final document.
Recompute from the current canonical root and relation state and apply the
existing source-scoped fences.

Resolve affected roots according to the relation lifecycle:

- A root create or update recomputes that root. If its relation keys change,
  update the corresponding relation indexes as part of normal root processing.
- Snapshot data has no independent trigger; its values are refreshed by the
  root event that carries the snapshot.
- An owned-child create, update, or delete recomputes its owning root, subject
  to bounded parent resolution and the root deletion-fence rules.
- A mutable independent-entity update uses the reverse index to durably schedule
  recomputation for every affected root under ADR-0057. Multi-hop dependencies
  propagate through the declared acyclic reverse-index edges until the affected
  roots are reached. Use ADR-0058's live/deferred fan-out path.
- If a required reverse index is incomplete or uncertain, repair or rebuild it
  before publishing recomputed values. A missing index entry does not prove
  there are no dependent roots.
- An unrelated source entity, a Redis cache refresh/eviction, or transport-only
  metadata does not by itself trigger projection recomputation.
- A manifest or transformation semantic change follows the schema-evolution
  and migration contract; it is not an ordinary source-event invalidation.

Independent-reference deletion does not cascade to roots. Its effect on copied
fields remains unresolved under Q-126, so production relations that depend on
this deletion case stay blocked.

For example, suppose a task projection includes `agency_name` from an agency
and `total_minutes` computed from its shifts. A shift duration change recomputes
its owning task; an agency update recomputes every task that references it. In
v1, an agency `internal_note` update also schedules those tasks even if
`internal_note` is not projected. A change to an unrelated building entity
triggers nothing. If a referenced agency is deleted, Q-126 still blocks that
production case rather than guessing whether to clear or retain
`agency_name`.

## Alternatives considered

1. **Entity/relation-level invalidation (selected).** It is straightforward to
   audit against the declared relation graph and does not depend on complete
   field-diff metadata. It can do extra work for unused-field changes.
2. **Field-level invalidation.** Explicitly declare which source paths each
   output depends on, then recompute only when an event changes one of those
   paths. This reduces work but duplicates transformation knowledge in the
   manifest and risks missed invalidation when declarations or change metadata
   are incomplete. Uncertain change descriptions would still require full
   relation-level recomputation.
3. **Recompute on cache/index changes.** This appears responsive to lookup state
   but makes derived Redis state a source of business invalidation. Cache repair
   and eviction must remain distinct from authoritative source changes.

## Consequences

- Manifests do not need per-output-field dependency declarations for v1.
- Correctness relies on the declared relation graph and complete/repaired
  reverse indexes; recomputation remains bounded through ADR-0057 and ADR-0058.
- Updates to unused fields can add write and recomputation load. Coalescing and
  deferred high-fan-out work mitigate bursts; benchmarks must expose the cost.
- Field-level invalidation can be considered later if measurements show the
  additional complexity is worthwhile.
- Independent-reference deletion remains separately gated by Q-126.

## Validation

- A root event recomputes only that root and updates relation indexes when keys
  change.
- An owned-child event recomputes its owner and cannot bypass root deletion
  fences.
- An independent-entity update recomputes every reverse-indexed dependent root;
  unrelated roots remain unchanged.
- A change to an unused field on a participating entity still triggers
  recomputation, while an unrelated source entity does not.
- Multi-hop invalidation reaches all declared dependent roots through complete
  reverse indexes; an incomplete index enters repair/rebuild before publication.
- Cache refresh/eviction does not trigger business recomputation.
- Independent-reference deletion remains blocked until Q-126 is resolved.

## Review triggers

Revisit if extra recomputation materially harms freshness or capacity, if source
change metadata proves complete and field-level dependencies can be validated
reliably, or if multi-hop reverse-index maintenance cannot establish
completeness.

## Related concepts

- [Transformation contract](../02-contracts/transformation-contract.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Reference change propagation](0057-reference-change-propagation.md)
- [Reference fan-out execution threshold](0058-reference-fanout-execution-threshold.md)
- [Relation lifecycle classification](0059-relation-lifecycle-classification.md)
- [Follow-up register](../follow-ups.md) (Q-017, Q-018, Q-126)
