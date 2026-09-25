---
type: Architecture Review Topic
title: Relationship model
description: Defines supported joins and dependency propagation across source entities.
tags: [contracts, relationships, joins, fanout]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0006
---

# Relationship model

## Decision

V1 supports explicitly declared, rooted, acyclic projection graphs with bounded
cardinality and a declared root-resolution strategy. Acyclic applies to
relationships that expand or resolve data into the projection, not to scalar
cross-reference IDs.

## Accepted relationship scope

- Root entity fields.
- Cached many-to-one references.
- Bounded one-to-many nested children.
- Explicitly declared multi-hop relations resolved through a cache or reverse
  index.

Every non-root relation declares its source topic and identity, parent/root
resolution strategy, cardinality and target path, update/delete behavior,
maximum expected fan-out, and rebuildability of required caches or reverse
indexes.

Every non-root relation also declares exactly one lifecycle role: `snapshot`,
`owned_child`, or `independent_entity`. This is a statement about the relation's
identity, event, and delete semantics in this projection; it does not choose its
Elasticsearch storage shape. See [ADR-0059](../decisions/0059-relation-lifecycle-classification.md).

Manifest validation rejects a missing or ambiguous lifecycle role, cycles in
the expansion graph, ambiguous parent resolution, and unbounded fan-out. A
scalar cross-reference such as
`related_task_id` is allowed because it does not recursively expand the related
entity.

Self-referential or mutually referential expansions are outside v1. Supporting
them requires a separate decision covering bounded depth, cycle detection, and
truncation semantics.

## Referenced-entity change propagation

When a mutable referenced entity changes or is deleted, use the maintained
reverse index from that entity to the root projection entities that depend on
it. Record durable recomputation work for those roots and process it in bounded
batches. Recompute from the current root and relation state, and apply
relation-derived values under the contributor-scoped source revision fence so
an older reference event cannot overwrite a newer value
([ADR-0057](../decisions/0057-reference-change-propagation.md),
[ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).

If the reverse index is missing or its completeness is uncertain, repair or
rebuild it before publishing based on its results; a missing entry does not
prove that no roots depend on the referenced entity. The numerical fan-out
ceiling for live recomputation is relation-specific and established by
representative benchmarks. At or below that ceiling, process the fan-out in
bounded live batches. Above it, preserve the change as durable deferred
recomputation work; the threshold selects a processing path and does not make
valid source data rejectable ([ADR-0058](../decisions/0058-reference-fanout-execution-threshold.md)).
The projection author's expected fan-out remains a planning and alert value, not
the measured live ceiling. Numeric live ceilings remain evidence-gated by Q-070.
The hard safety envelope remains separate and follows ADR-0056. This decision
covers changes to referenced entities; root events that change their own
relation keys continue through normal root recomputation and relation-index
maintenance.

V1 invalidates at the source-entity/relation level: every accepted create/update
for an entity participating in the materialized dependency graph, every
owned-child delete, and every independent-reference delete schedules affected
root recomputation under that relation's lifecycle. Recompute even if the
changed field is not copied into the projection. For an independent-reference
delete, remove optional copied fields or apply an explicitly safe missing-value
rule; do not cascade-delete the root. Resolve impacts through the declared
relation graph and repair an incomplete reverse index before publishing.
Field-level invalidation is not required in v1
([ADR-0061](../decisions/0061-projection-dependency-invalidation.md)).

## Relation lifecycle classification

- **Snapshot**: embedded data with no independent identity or separately fenced
  event lifecycle in this projection. It follows the root's updates and deletes.
- **Owned child**: a separately identified member that belongs to one root's
  projection lifecycle. Its events update or remove only that member. Deleting
  the root removes the projection; a later child event cannot resurrect a root
  protected by its deletion fence. If no deletion fence is known, child-before-
  parent handling follows the bounded context-resolution and orphan rules in
  the stream contract. The engine never deletes source records as a side effect
  of projection deletion.
- **Independent entity**: a separately identified source entity with its own
  lifecycle that may contribute to multiple roots. Changes propagate to
  affected roots under ADR-0057 and ADR-0058. Deleting it does not delete those
  roots; recomputation removes optional copied fields or applies an explicitly
  safe missing-value rule ([ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).

These roles have different observable outcomes. For example, a task's embedded
display preferences can be a snapshot: the next task update supplies their
current values, and there is no separate preference delete event to fence. A
shift with its own ID and CDC events can be an owned child: deleting shift `S7`
removes only `S7`, while a delayed event for `S7` cannot resurrect a task whose
deletion fence is still valid. An agency referenced by 100 tasks is independent:
an agency-name change recomputes those dependent task projections, and deleting
one task does not delete the agency. If the agency is deleted, dependent tasks
remain; optional `agency_name` fields are removed, while an explicitly declared
safe missing-value rule may provide a replacement.

Using one role for all three would impose the wrong lifecycle rule somewhere:
treating every relation as a snapshot cannot safely process independently
ordered child events; treating every relation as owned would give a shared
agency many false owners and could couple its projection to one task's deletion;
treating every relation as independent would require standalone identity,
fencing, and propagation machinery for embedded values such as preferences.
Lifecycle role and physical layout (nested member, embedded object, or separate
index) therefore remain separate manifest choices.

## Pros

- Covers the examples without pretending to be a general graph engine.
- Makes reverse-index and root-resolution requirements explicit.
- Allows useful scalar links without recursive projection complexity.
- Validation can reject relationships the runtime cannot update safely.

## Cons and risks

- Reference updates such as organization changes may fan out to many roots.
- Cached reverse maps can be large and expensive to rebuild.
- Limiting expansion shapes may exclude some self-referential projections.
- Bounded recursion, if later added, will require additional state and limits.

## Questions to stamp

- **Resolved by [ADR-0057](../decisions/0057-reference-change-propagation.md):** How are reference-field changes propagated to existing root documents? Use the maintained reverse index to durably schedule bounded recomputation of affected roots, fence relation-derived writes by contributor revision, and repair an incomplete index before publishing from it.
- **Resolved by [ADR-0058](../decisions/0058-reference-fanout-execution-threshold.md):** What fan-out limit triggers deferred rebuild instead of live updates? Use a benchmark-derived live-update ceiling per relation; larger fan-outs use durable deferred recomputation, with numeric ceilings tracked by Q-070.
- **Resolved by [ADR-0059](../decisions/0059-relation-lifecycle-classification.md):** Are relations snapshots, owned children, or independent entities? Every non-root relation declares one role, with separate identity, update, and delete semantics; this does not select physical storage. An owned-child event cannot resurrect a root protected by its deletion fence.
- **Resolved by [ADR-0061](../decisions/0061-projection-dependency-invalidation.md):** Which dependency changes trigger recomputation? Invalidate at entity/relation granularity across the declared acyclic dependency graph, including changes to unused fields on participating entities; do not use incomplete reverse indexes to conclude there are no dependents.
- **Resolved by [ADR-0063](../decisions/0063-independent-reference-delete-effects.md):** What happens to copied fields when an independent reference is deleted? Recompute reverse-indexed roots, remove optional derived fields or apply explicit safe missing-value behavior, and do not cascade-delete roots.

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Identity, time, and ordering](identity-time-ordering.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Deletion and replay](deletion-and-replay.md)
