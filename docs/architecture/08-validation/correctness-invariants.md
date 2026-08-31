---
type: Architecture Review Topic
title: Correctness invariants
description: Defines the properties that implementations must preserve across duplicates, disorder, and failure.
tags: [validation, correctness, invariants]
status: accepted
decision_id: ADR-0032
accepted_on: 2026-08-31
owner: TBD
conditions:
  - The narrowest-safe-scope rule is the default response to invariant violations.
  - Canonical comparisons support an explicit unknown result when a required source boundary is unavailable.
  - The exact shared-metadata failures that stop all projections are deferred to a later decision.
  - Conditional convergence claims name their retention, authority, boundary, and manifest preconditions.
---

# Correctness invariants

## Decision

Define correctness in three classes.

### Absolute safety invariants

These must never be violated:

- A committed Kafka offset has a definitive outcome for every required target.
- An older source revision cannot overwrite a newer revision within its source
  scope.
- An older update cannot resurrect a deleted entity protected by a deletion
  fence.
- One logical identity cannot produce conflicting Elasticsearch document
  identities.
- A read alias never points to an unverified migration target.
- Identical canonical source inputs produce identical projection content.
- A privileged operation is attributable and durably recorded.

Event-local ambiguity or malformed data receives a durable DLQ terminal outcome.
Projection- or partition-scoped safety uncertainty pauses or blocks the
narrowest safe scope. Shared metadata, fencing, or custody corruption is treated
as unsafe, but the exact conditions that stop all projections are deferred to a
later decision.

### Conditional convergence invariants

These hold when their documented preconditions are available:

- Bootstrap plus boundary overlap replay converges to MongoDB-derived source
  truth.
- Redis and Elasticsearch can be rebuilt from authoritative MongoDB state plus
  sufficient CDC history.
- Reconciliation repair restores the canonical projection through the normal
  fenced path.
- A child-before-parent event eventually converges when the parent becomes
  available or is explicitly classified as missing.

Each claim names its preconditions, including retained CDC history, accessible
MongoDB authority, valid manifests, source boundaries, and sufficient
deletion/fence metadata.

### Bounded operational properties

Freshness, recovery time, cache staleness, retry completion time, and bootstrap
duration are measured objectives rather than absolute correctness invariants.
Temporary degradation does not imply corrupt data when the absolute safety
invariants remain intact.

### Canonical source state

Use MongoDB-derived, entity-level canonical state evaluated with the pinned
manifest and transformation version. Do not claim a globally atomic snapshot
across independent MongoDB services unless such a snapshot actually exists.

For cross-service relationships, comparisons record source boundaries, observed
versions, missing dependencies, and fence state. A comparison may return
`unknown` when a required source boundary or authoritative input is unavailable;
`unknown` is not treated as a mismatch and does not trigger repair by itself.

## Pros

- Makes safety claims strict while keeping convergence claims honest and
  conditional.
- Gives tests and operations a shared oracle independent of implementation.
- Prevents a single event-local problem from stopping unrelated work.
- Makes unavailable source evidence distinguishable from actual drift.

## Cons and risks

- The system needs richer status and validation metadata.
- Choosing the narrowest safe scope can leave some work paused longer while the
  affected boundary is investigated.
- Conditional convergence depends on retention and source availability being
  monitored.
- The deferred global-stop threshold remains an operational risk until decided.

## Alternatives considered

1. Treat every invariant as absolute. This overstates guarantees across outages
   and independently changing MongoDB services.
2. Use only final document comparisons. This misses offset custody, stale writes,
   deletion resurrection, and unsafe migration transitions.
3. Stop the entire engine for every violation. This unnecessarily turns
   event-local or projection-specific failures into global outages.
4. Turn every violation into a DLQ record. This can hide corrupted metadata or
   fencing and permit unsafe progress.

## Consequences

- Validation must distinguish `match`, `mismatch`, and `unknown` outcomes.
- Every convergence result records the authority, boundary, manifest, and
  transformation assumptions used.
- Failure handling needs an explicit narrowest-safe-scope decision tree.
- Global-stop behavior for shared metadata failures remains a follow-up rather
  than an implicit implementation choice.

## Validation

- Model and property tests cover duplicate, reordered, delayed, deleted, and
  replayed histories.
- Tests prove stale updates and delete resurrection cannot win.
- Crash tests prove committed offsets always have terminal custody.
- Bootstrap and migration tests prove convergence and verified cutover under
  their documented preconditions.
- Reconciliation tests prove repairs use the normal fenced path and do not act on
  `unknown` comparisons alone.
- Failure tests prove event-local, projection-scoped, and global handling occurs
  at the intended scope.

## Review trigger

Revisit if a required invariant cannot be tested, source boundaries or retention
cannot support a convergence claim, `unknown` results accumulate without
resolution, or shared metadata incidents expose the need for a global-stop
threshold.

## Related concepts

- [Correctness invariants](correctness-invariants.md)
- [Identity and ordering](../02-contracts/identity-time-ordering.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Test strategy](test-strategy.md)

## Follow-up questions

- Which shared metadata failures justify stopping all projections immediately?
- How are unresolved `unknown` comparisons aged, alerted, and eventually
  resolved?
- Which convergence preconditions are release-blocking versus operational
  warnings?
