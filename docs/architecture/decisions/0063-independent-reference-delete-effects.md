---
type: Architecture Decision Record
title: "ADR-0063: Independent-reference delete effects"
description: Defines how dependent projections change when a referenced entity is deleted.
tags: [architecture, adr, deletion, relationships, recomputation]
status: accepted
decision_id: ADR-0063
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Deleting an independent referenced entity recomputes affected roots but never deletes those roots.
  - Remove copied fields declared optional, or apply the manifest's explicit safe missing-value behavior.
  - Block a manifest before production if a required output depends on the reference and has no safe missing-value behavior.
  - Resolve affected roots from complete reverse indexes and use the existing bounded fan-out and fencing rules.
---

# ADR-0063: Independent-reference delete effects

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

An independent entity can contribute fields to many root documents. The
accepted lifecycle rule says its deletion does not cascade to those roots, but
the engine still needs to decide what happens to fields already copied from it.
Keeping the old value can leave stale data in Elasticsearch; deleting the whole
root removes data that belongs to an independent source entity.

The existing transformation contract already requires output fields to be
declared optional before they can be omitted and requires an explicit safe rule
before a transformation can publish with missing context
([ADR-0060](0060-bloblang-subset-and-resource-budgets.md)). Reference changes
already use reverse-indexed recomputation and bounded high-fan-out work
([ADR-0057](0057-reference-change-propagation.md),
[ADR-0058](0058-reference-fanout-execution-threshold.md)).

## Decision

Treat an independent referenced-entity delete as a dependency invalidation. Use
the complete reverse index to durably schedule recomputation for affected roots,
and fence relation-derived writes by the deleted entity's source revision. Do
not delete dependent root documents or unrelated root fields. This extends the
invalidation triggers in
[ADR-0061](0061-projection-dependency-invalidation.md) and resolves the copied-
field question left open by
[ADR-0059](0059-relation-lifecycle-classification.md).

Recompute each affected root with the deleted reference known to be absent:

- Remove fields derived only from that reference when the manifest declares
  those outputs optional.
- Apply a deterministic missing-value behavior only when the manifest
  explicitly declares it safe, as required by ADR-0060.
- Before production, block a manifest when a required output depends on an
  independent reference but has no safe missing-value behavior. Do not wait for
  a delete event to discover that its configured transformation cannot produce
  a complete document.
- If runtime state still prevents safe recomputation, durably defer or repair
  the affected work and expose its pending/blocked status. Do not DLQ a valid
  delete or report affected roots as refreshed. Repair an incomplete reverse
  index before publishing any result from it.

The same bounded fan-out policy remains in force. Process a delete's affected
roots in bounded live work when they are within the relation's measured ceiling;
route larger fan-out to durable deferred recomputation under ADR-0058. A valid
delete is not rejected because its fan-out is large.

For example, agency `A17` contributes `agency_name: "North branch"` to 100 task
documents. On deletion, the engine retains all 100 task roots and schedules
their recomputation. If `agency_name` is optional, each task loses that field
while retaining its own title and status. A declared safe default may supply a
replacement. If `agency_name` is mandatory and has no safe missing-value rule,
preflight blocks that projection before production rather than leaving the
engine to guess when the agency is deleted.

## Alternatives considered

1. **Keep the last copied value.** This preserves display continuity and avoids
   immediate fan-out work, but the value can remain stale indefinitely and may
   conflict with source deletion or anonymization.
2. **Cascade-delete every dependent root.** This removes copied data, but
   destroys independently owned documents and is incorrect when a shared
   reference contributes to many roots.
3. **Require an operator to choose a field outcome for each delete.** This
   avoids an automatic guess, but makes ordinary source lifecycle events
   operational incidents and can hold valid work until a person intervenes.
4. **Recompute using declared optionality or safe missing-value behavior
   (selected).** This makes the outcome deterministic at manifest preflight and
   keeps roots independent. It requires projection authors to declare how
   required derived values behave when a reference is absent.

## Consequences

- Deleting a shared reference updates only roots found through its reverse
  index; it never cascades into root deletion.
- Optional copied values are removed, so a deleted source value does not remain
  in a successfully recomputed target document.
- A manifest with a required reference-derived output must provide an explicit
  safe missing-value rule before production; this moves ambiguity to preflight.
- Large fan-outs may converge asynchronously under ADR-0058. While valid work
  is pending, the engine must expose the affected scope as not yet refreshed.
- Recompute cost and temporary target staleness remain risks during dependency
  outages; they do not justify DLQing or discarding a valid source delete.

## Validation

- Deleting an independent reference schedules only its dependent roots and
  preserves each root and its unrelated fields.
- An optional copied field is removed; an explicitly safe missing-value rule
  produces the declared result.
- Preflight blocks a required reference-derived output without a safe
  missing-value rule.
- A delete with fan-out above the live ceiling becomes durable deferred work,
  not a rejected event or DLQ record.
- An incomplete reverse index is repaired before publishing recomputed roots.
- A stale reference update cannot restore deleted content over the newer delete
  fence.

## Review triggers

Revisit if consumers require last-known reference values, if source deletion
must erase derived values immediately rather than through normal CDC
convergence, or if relation fan-out cannot meet freshness and capacity
objectives.

## Related decisions and concepts

- [Relation lifecycle classification](0059-relation-lifecycle-classification.md)
- [Projection dependency invalidation](0061-projection-dependency-invalidation.md)
- [Reference change propagation](0057-reference-change-propagation.md)
- [Reference fan-out execution threshold](0058-reference-fanout-execution-threshold.md)
- [Bloblang subset and resource budgets](0060-bloblang-subset-and-resource-budgets.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Follow-up register](../follow-ups.md) (Q-126)
