---
type: Architecture Decision Record
title: "ADR-0051: Rebuilding child membership from source snapshots"
description: Defines when a source snapshot may establish that a nested child is absent during a projection rebuild.
tags: [architecture, adr, deletion, bootstrap, snapshots, contracts]
status: accepted
decision_id: ADR-0051
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Snapshot absence is authoritative only when the relation was completely enumerated at a recorded source boundary.
  - A complete snapshot omits absent children from the rebuilt projection but does not invent a child-level source revision or deletion fence.
  - Rebuild handoff remains pending until ordering evidence prevents older changes from restoring absent children; ambiguous completeness or ordering blocks cutover and writes.
---

# ADR-0051: Rebuilding child membership from source snapshots

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

A rebuilt projection can omit a nested child because the source no longer
contains it. That absence is meaningful only if the snapshot covers the full
relation at a known source boundary. A partial scan, lagging read, or ambiguous
source boundary could omit a child that still exists. Conversely, preserving
children from an older target can retain deleted data in a new projection.

Bootstrap also overlaps snapshot work with CDC delivery. A child update from
before the snapshot boundary could arrive after the snapshot write. The exact
source fields and comparison scope that reject such stale changes remain open
in Q-007.

## Decision

- Treat child absence as authoritative only after the relation is completely
  enumerated at a recorded source boundary. Relation completeness must be known
  for the specific snapshot input; a partial page, failed scan, or unavailable
  source is not evidence of absence.
- When a complete source-bounded snapshot omits a child, omit that child from
  the rebuilt projection. Do not copy the child forward from an older target.
- Do not manufacture a child-level source revision or deletion fence from
  absence alone. Preserve any durable child deletion fence that already exists,
  and process boundary-overlap changes through the normal fenced path.
- Keep the rebuild pending and block target cutover or writes if snapshot
  completeness is unknown or available ordering evidence cannot show that an
  older child change will be rejected. Resolve the ambiguity using an
  authoritative source read or reconciliation; do not guess.
- The precise connector ordering fields and comparison scope remain governed
  by Q-007. This ADR defines the snapshot rule without pre-empting that
  decision.

## Alternatives considered

1. **Trust any snapshot omission.** This is simple and fast, but a partial or
   stale scan can silently remove a child that still exists in the source.
2. **Keep children from the previous target whenever the snapshot omits them.**
   This avoids deletion based on uncertain absence, but carries stale or deleted
   data into a rebuild and makes the old target an undeclared source of truth.
3. **Require complete enumeration and a source-boundary handoff.** This adds
   completeness tracking and can delay cutover while ordering evidence is
   gathered, but makes absence meaningful and fails closed when it is not.
   This is the accepted approach.

## Consequences

- Snapshot or relation readers must expose whether enumeration completed for
  the relation and source boundary used by the rebuild.
- A complete snapshot can remove a child from a rebuilt document even if no
  delete event is available, while existing fences and boundary replay still
  protect against older changes.
- Partial snapshots and unresolved ordering keep the rebuild visibly pending
  instead of producing a target that may be missing or resurrecting children.
- Q-007 remains a prerequisite for choosing the exact source fields that prove
  an overlap event is older than the snapshot boundary.

## Validation

- A complete source-bounded snapshot omitting a child produces a rebuilt
  document without that child.
- An incomplete or failed relation scan does not treat omission as deletion and
  cannot complete the rebuild or cut over the target.
- A pre-boundary child change cannot restore a child omitted by the snapshot;
  if the ordering evidence is unavailable, the rebuild remains pending.
- A valid newer child change after the boundary can add the child again through
  the normal fenced path.
- Any retained child deletion fence remains effective during snapshot and
  boundary-overlap processing.

## Review triggers

Revisit if a source cannot enumerate the full relation at a boundary, the
connector's ordering fields cannot reject stale overlap changes, a new snapshot
mode is introduced, or production evidence shows false child removals or
resurrections.

## Related decisions and concepts

- [ADR-0008: Deletion and replay semantics](0008-deletion-and-replay-semantics.md)
- [ADR-0015: Bootstrap consistency and handoff](0015-bootstrap-consistency-and-handoff.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Follow-up register](../follow-ups.md) (Q-006 and Q-007)
