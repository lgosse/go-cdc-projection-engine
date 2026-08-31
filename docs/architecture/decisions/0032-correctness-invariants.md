---
type: Architecture Decision Record
title: "ADR-0032: Correctness invariants"
description: Correctness uses strict safety invariants, conditional convergence claims, and narrowest-safe-scope failure handling.
tags: [architecture, adr, validation, correctness, invariants]
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

# ADR-0032: Correctness invariants

## Context

The accepted runtime, deletion, bootstrap, migration, and reconciliation
decisions need a shared oracle before test strategy and release gates can be
defined. The source systems change independently, Kafka delivery is at-least-
once and unordered across topics, and outages can temporarily remove evidence
without proving that data is wrong.

## Decision

Define correctness in three classes.

Absolute safety invariants must never be violated:

- every committed Kafka offset has a definitive outcome for every required
  target;
- older source revisions cannot overwrite newer revisions within their source
  scope;
- older updates cannot resurrect entities protected by deletion fences;
- one logical identity cannot produce conflicting Elasticsearch document
  identities;
- read aliases never point to unverified migration targets;
- identical canonical source inputs produce identical projection content; and
- privileged operations are attributable and durably recorded.

Handle violations at the narrowest safe scope by default. Event-local ambiguity
or malformed data receives a durable DLQ terminal outcome. Projection- or
partition-scoped safety uncertainty pauses or blocks that scope. Shared metadata,
fencing, or custody corruption is unsafe, but the exact conditions that stop all
projections are deferred to a later decision.

Conditional convergence invariants hold only when documented preconditions are
available: retained CDC history, accessible MongoDB authority, valid manifests,
source boundaries, and sufficient deletion/fence metadata. These include
bootstrap plus overlap replay, Redis and Elasticsearch rebuilds, reconciliation
repair, and eventual child-before-parent convergence.

Freshness, recovery time, cache staleness, retry completion time, and bootstrap
duration are bounded operational objectives rather than absolute correctness
claims.

Canonical state is MongoDB-derived, entity-level state evaluated with the pinned
manifest and transformation version. No globally atomic snapshot is claimed
across independent MongoDB services without an actual shared snapshot.
Comparisons record source boundaries, observed versions, missing dependencies,
and fence state. A comparison returns `unknown` when a required source boundary
or authoritative input is unavailable; `unknown` is not a mismatch and does not
trigger repair by itself.

## Alternatives considered

1. **Treat every invariant as absolute.** This overstates guarantees across
   outages and independently changing MongoDB services.
2. **Use only final document comparisons.** This misses offset custody, stale
   writes, deletion resurrection, and unsafe migration transitions.
3. **Stop the entire engine for every violation.** This unnecessarily turns
   event-local or projection-specific failures into global outages.
4. **Turn every violation into a DLQ record.** This can hide corrupted metadata
   or fencing and permit unsafe progress.

## Consequences

- Validation distinguishes `match`, `mismatch`, and `unknown` outcomes.
- Convergence results record authority, boundary, manifest, and transformation
  assumptions.
- Failure handling follows an explicit narrowest-safe-scope decision tree.
- Global-stop behavior for shared metadata failures remains a follow-up.

## Validation

- Model and property tests cover duplicate, reordered, delayed, deleted, and
  replayed histories.
- Tests prove stale updates and delete resurrection cannot win.
- Crash tests prove committed offsets always have terminal custody.
- Bootstrap and migration tests prove convergence and verified cutover under
  documented preconditions.
- Reconciliation tests prove repairs use the normal fenced path and do not act
  on `unknown` comparisons alone.
- Failure tests prove event-local, projection-scoped, and global handling occurs
  at the intended scope.

## Review triggers

Revisit if a required invariant cannot be tested, source boundaries or retention
cannot support a convergence claim, `unknown` results accumulate without
resolution, or shared metadata incidents expose the need for a global-stop
threshold.

## Related concepts

- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Identity and ordering](../02-contracts/identity-time-ordering.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Test strategy](../08-validation/test-strategy.md)
