---
type: Architecture Decision Record
title: "ADR-0059: Relation lifecycle classification"
description: Defines snapshot, owned-child, and independent-entity relation lifecycle semantics.
tags: [architecture, adr, relationships, lifecycle]
status: accepted
decision_id: ADR-0059
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Every non-root manifest relation declares exactly one lifecycle role.
  - Lifecycle role is separate from Elasticsearch storage shape.
  - Owned-child events cannot resurrect a root protected by its deletion fence or delete source records.
  - Independent-entity deletion does not cascade to dependent roots; copied-field behavior remains Q-126.
---

# ADR-0059: Relation lifecycle classification

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

The engine projects root entities together with related source data. Those
relations do not all have the same identity or lifecycle. Some values arrive
only as part of a root snapshot, some have their own identity but belong to one
root, and some are shared source entities that contribute to several roots.
Treating all three alike would make event ordering, deletion, and propagation
incorrect or needlessly expensive.

The lifecycle classification must also remain independent of the physical
Elasticsearch layout. A lifecycle role says how source events and deletion
behave; nested arrays, embedded objects, and separate indices describe where
the projection is stored.

## Decision

Every non-root relation in a manifest declares exactly one lifecycle role:

- **Snapshot**: related data is embedded without its own identity or separately
  fenced event lifecycle in this projection. It follows root updates and
  deletion.
- **Owned child**: a separately identified member belongs to one root's
  projection lifecycle. Its event updates or deletes only that member. A late
  child event cannot resurrect a root protected by a retained deletion fence.
  When no such fence is known, child-before-parent resolution follows the
  bounded context-resolution and orphan rules in ADR-0013. A valid root event
  newer than the deletion fence may recreate the root. Deleting a root removes
  the projection but never deletes source records.
- **Independent entity**: a separately identified entity has its own lifecycle
  and may contribute to multiple roots. Changes propagate to affected roots
  under ADR-0057 and ADR-0058. Deleting the entity does not delete dependent
  roots. The effect on fields already copied to those roots remains open under
  Q-126; until decided, production relations that depend on this delete case
  remain blocked, with no implicit cascade or field mutation.

Lifecycle role does not select physical storage. The manifest and projection
capacity contract separately choose an embedded object, nested member, or
separate index.

For example, a task's display preferences can be a snapshot if they have no
separate identity or event stream. A shift with its own ID and CDC events can be
an owned child: deleting shift `S7` removes only `S7`, and a delayed event cannot
resurrect a task protected by its deletion fence. An agency referenced by 100
tasks is an independent entity: an agency-name change recomputes those dependent
task projections, but deleting one task does not delete the agency. What happens to the 100 tasks'
copied agency fields when the agency itself is deleted is still Q-126.

## Alternatives considered

1. **Treat every relation as a snapshot.** This is simple for embedded values,
   but cannot safely represent an independently ordered shift-delete event or
   fence that member against stale replay.
2. **Treat every relation as an owned child.** This invents a single owner for
   shared entities such as an agency. Deleting one task could then incorrectly
   couple the agency's projected lifecycle to that task, and the same agency
   would have many conflicting owners.
3. **Treat every relation as independent.** This would require identity,
   fencing, and propagation machinery for values such as embedded display
   preferences that have no separate source lifecycle.
4. **Let each relation's behavior be inferred from storage shape.** This
   conflates Elasticsearch representation with source lifecycle and makes the
   same source semantics vary when an index layout changes.

## Consequences

- Manifest validation rejects a missing or ambiguous lifecycle role.
- Event handling and rebuilds use the declared role to decide identity,
  fencing, membership, and delete behavior.
- Source deletes and projection deletes remain separate actions; the engine
  never deletes source records as a projection side effect.
- Independent-entity change propagation uses the accepted reverse-indexed
  recomputation path. Independent-entity delete handling remains incomplete
  until Q-126 is resolved.
- Storage and capacity decisions remain independently reviewable.

## Validation

- Reject manifests with a missing or invalid role for any non-root relation.
- Verify root updates refresh snapshots without creating an independent fence.
- Verify an owned-child update/delete affects only that member, and a late child
  event cannot resurrect a root protected by its deletion fence. Ordinary
  child-before-parent cases still follow ADR-0013.
- Verify a shared independent-entity update recomputes its dependent roots and
  deleting a root does not delete that shared source entity.
- Verify changing nested-versus-separate-index storage does not change lifecycle
  behavior.
- Keep the independent-reference-delete production case blocked until Q-126
  defines copied-field behavior.

## Review triggers

Revisit if a source relation cannot be classified unambiguously, if a relation
needs both owned and shared lifecycle semantics, or if new storage modes cause
the lifecycle rules to diverge.

## Related concepts

- [Relationship model](../02-contracts/relationship-model.md)
- [Domain language](../01-system-context/domain-language.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Reference change propagation](0057-reference-change-propagation.md)
- [Reference fan-out execution threshold](0058-reference-fanout-execution-threshold.md)
- [Follow-up register](../follow-ups.md) (Q-015, Q-016, Q-126)
