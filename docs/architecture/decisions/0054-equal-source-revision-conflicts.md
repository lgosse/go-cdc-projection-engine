---
type: Architecture Decision Record
title: "ADR-0054: Equal source revision conflicts"
description: Defines duplicate and conflict handling when one entity receives the same MongoDB source revision more than once.
tags: [architecture, adr, cdc, mongodb, ordering, correctness]
status: accepted
decision_id: ADR-0054
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - For one scoped entity, an equal source revision with the same logical source mutation is a duplicate and is a no-op.
  - Different source mutations at the same revision are an integrity conflict; no arrival, Kafka offset, or connector-time tie-breaker is allowed.
  - Preserve current projection state, durably retain the conflict before offset completion, and hold that entity for authoritative repair.
  - The source contract expects different mutations to have different revisions, but production connector behavior remains an evidence gate under Q-123.
---

# ADR-0054: Equal source revision conflicts

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

[ADR-0052](0052-mongodb-source-ordering-scope.md) defines the MongoDB source
revision as `(source.ts_ms, source.ord)` within one source, replica set,
database, collection, and canonical entity ID. The intended contract is that a
revision identifies one source mutation. At-least-once delivery can repeat that
mutation, so seeing an equal revision is not by itself an error. Two different
mutations for the same entity at the same revision, however, have no source
ordering that would establish which content is newer.

The reviewed development event and connector configuration do not prove that
the deployed production plugin always provides the required ordering behavior.
Production verification remains an enablement gate under Q-123.

For example, if updates for one entity both carry `(source.ts_ms,
source.ord) = (1700000000000, 7)`, and both say `status: "active"`, the second
delivery is a duplicate. If one says `status: "active"` and the other says
`status: "inactive"`, neither tuple establishes which content is newer.

## Decision

- For the same scoped entity and source revision, treat a repeated delivery of
  the same logical source mutation as a duplicate and perform no projection
  write. Compare the decoded source mutation; envelope processing timestamps,
  Kafka coordinates, and other delivery metadata do not make it a different
  source mutation.
- If the same entity and revision arrive with different source mutations,
  classify this as a source/connector integrity conflict. Do not choose a winner
  using arrival order, Kafka partition/offset, outer event `ts_ms`, or a
  lexicographic/hash ordering of payloads.
- Keep the current projection and fence unchanged. Durably retain the conflicting
  event before completing its delivery checkpoint, mark the entity for repair,
  and do not apply further events to that entity until an authoritative repair
  re-establishes its current state and fence. Unrelated entities may continue.
- Treat uniqueness of the source revision for distinct mutations as an expected
  source-contract invariant, not as already verified production evidence.
  Verify the deployed plugin and representative event metadata before enabling
  a production source (Q-123).

## Alternatives considered

1. **Use the later Kafka offset or arrival as the winner.** This is simple, but
   delivery position is not source freshness and may change during replay or
   repartitioning.
2. **Use outer event time or payload ordering as a tie-breaker.** Connector
   processing time does not establish source order; a payload hash or lexical
   comparison is deterministic but arbitrarily chooses business state.
3. **Treat equal revision and equal mutation as a duplicate; quarantine
   conflicting mutations for repair.** This preserves idempotence for ordinary
   redelivery and avoids silently inventing an order for an integrity anomaly.
   It can leave one entity unavailable until repair, so the hold is entity-local.
   This is accepted.

## Consequences

- Ordinary redelivery remains idempotent without a new ordering field.
- A conflicting equal-revision event cannot silently overwrite state. It adds
  durable custody and repair work and temporarily blocks writes for that entity.
- The engine expects different source mutations to have different revisions,
  but does not claim that production has proved this invariant yet.
- No tie-breaker is added to the source revision tuple. If a future source has
  legitimate same-revision mutations, it must provide a stronger revision or
  receive a separately reviewed ordering policy.

## Validation

- Delivering the same source mutation twice at the same scoped revision results
  in one projection mutation; the repeat is a no-op.
- Delivering different source mutations for one entity at the same revision
  leaves the existing projection and fence unchanged, durably retains the
  conflict before checkpoint completion, and marks only that entity for repair.
- Events for the conflicted entity do not write until authoritative repair
  establishes current state and a valid fence; unrelated entities continue.
- Replaying the conflict does not select a winner based on Kafka offset,
  arrival order, outer event time, or payload hash.
- Q-123 verifies the deployed plugin and representative create, update, and
  delete metadata before production-source enablement; a sample is not treated
  as proof that collisions are impossible in all cases.

## Review triggers

Revisit if production evidence shows that distinct mutations can legitimately
share a revision, if repair cannot restore an entity without losing source
history, or if the source offers a stronger stable per-entity revision.

## Related decisions and concepts

- [ADR-0052: MongoDB source-ordering scope](0052-mongodb-source-ordering-scope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Follow-up register](../follow-ups.md) (Q-009, Q-123)
