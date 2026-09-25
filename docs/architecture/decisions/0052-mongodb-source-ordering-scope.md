---
type: Architecture Decision Record
title: "ADR-0052: MongoDB source-ordering scope"
description: Defines the source revision tuple and comparison scope for MongoDB CDC events.
tags: [architecture, adr, cdc, debezium, mongodb, ordering]
status: accepted
decision_id: ADR-0052
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Freshness uses source.ts_ms and source.ord, namespaced by source.name and source.rs, for the same database, collection, and canonical entity ID.
  - The tuple is compared lexicographically only within that stable source and entity scope; events from different replica sets or sources are incomparable.
  - Missing or incomparable source metadata is deferred or repaired, not applied using processing timestamps or Kafka offsets.
  - Production connector field availability and deployed plugin version must be verified before a production source is enabled (Q-123).
---

# ADR-0052: MongoDB source-ordering scope

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

The checked CDC example is a development event from Debezium `2.2.1.Final`.
It contains `source.ts_ms`, `source.ord`, `source.name`, `source.rs`, `source.db`,
and `source.collection`; its `source.lsid` and `source.txnNumber` are null. The
Terraform connector templates select `MongoDbConnector`, but the production
configuration declares a custom Kafka Connect image tag rather than the exact
Debezium plugin version. The example therefore supports a candidate contract,
but does not prove that the deployed production connector emits it.

Debezium documents `source.ts_ms` as the database change timestamp and `ord` as
the event ordinal within that timestamp. The top-level event `ts_ms` reports
connector processing time. `source.wallTime` is present in the checked event
example and was added to MongoDB source metadata in Debezium 2.2, but it remains
the bootstrap-boundary field described by [ADR-0015](0015-bootstrap-consistency-and-handoff.md),
not the per-entity freshness tuple.

## Decision

- Require `source.ts_ms`, `source.ord`, `source.name`, `source.rs`, `source.db`,
  `source.collection`, and canonical entity identity for source freshness.
- Namespace each entity fence by `(source.name, source.rs, source.db,
  source.collection, canonical_entity_id)`. Compare `(source.ts_ms, source.ord)`
  lexicographically only for that same entity within the same source and
  replica-set scope. A larger timestamp is newer; for the same timestamp, a
  larger ordinal is newer.
- Do not compare revisions across independent source names or replica sets. If
  an entity appears under a different `source.rs`, or the scope cannot be
  established, defer it for repair or rebuild from current authoritative source
  state; do not invent a cross-shard ordering.
- Do not use the outer event `ts_ms`, `source.wallTime`, Kafka partition/offset,
  `source.h`, or transaction metadata as substitutes for this per-entity tuple.
  `wallTime` retains its accepted bootstrap-boundary role; transaction
  metadata remains diagnostic context under ADR-0049.
- If any required field is absent, malformed, or incomparable, do not mutate
  the projection with that event. Keep it pending or route it through the
  accepted repair/DLQ policy. Equal revisions follow duplicate and
  conflict-repair rules in [ADR-0054](0054-equal-source-revision-conflicts.md).
- Treat production availability as an enablement gate, not an assumption. Verify
  the deployed plugin version and representative production `c`/`u`/`d` source
  fields before enabling a production connector (Q-123). This resolves Q-007's
  required contract and comparison scope; it does not claim that the present
  production image has already passed that evidence check.

## Alternatives considered

1. **Use only `source.ts_ms`.** This is easy to understand, but events can share
   a timestamp; without `ord`, the engine cannot distinguish their order.
2. **Use `source.wallTime` or the outer event `ts_ms`.** A timestamp is not a
   source-scoped sequence. The outer field reflects connector processing time,
   and neither field replaces the timestamp ordinal for freshness.
3. **Use Kafka partition and offset.** These are stable delivery coordinates
   within a partition, but do not compare source freshness across topics,
   partitions, or independent connector streams.
4. **Compare one tuple across an entire sharded cluster.** This would simplify
   fences but invents an order between independent replica sets that the chosen
   source tuple does not establish.

## Consequences

- A source that does not provide and preserve the required fields cannot be
  enabled until it has a safe adapter or an explicit later decision.
- The ordering contract is precise for one entity within one replica-set scope,
  but it does not promise cross-shard or cross-source total order.
- A source-scope change requires authoritative repair or rebuild rather than
  comparing incomparable tuples.
- Equal-revision conflicts do not use an arrival-order tie-break; identical
  source mutations are no-ops and conflicting mutations require repair
  ([ADR-0054](0054-equal-source-revision-conflicts.md)).

## Validation

- A later `source.ts_ms` wins for the same scoped entity; equal timestamps use
  the greater `source.ord`.
- Reordered delivery of the same source tuple does not allow an older tuple to
  overwrite newer state.
- The outer event `ts_ms`, Kafka offset, transaction fields, and `wallTime` do
  not change the freshness comparison.
- Missing tuple fields and changed source/replica-set scope remain pending or
  take the approved repair/rebuild path.
- Q-123 verifies the production image and representative create, update, and
  delete events before production-source enablement.
- Equal tuple with identical content is a duplicate no-op; conflicting content
  is retained and repaired under ADR-0054.

## Review triggers

Revisit if the Debezium plugin version or capture mode changes, a connector no
longer emits the required fields, entity movement across replica sets must be
supported without rebuild, or a source provides a stronger stable revision.

## Related decisions and concepts

- [ADR-0005: Identity and ordering scope](0005-identity-and-ordering-scope.md)
- [ADR-0015: Bootstrap consistency and handoff](0015-bootstrap-consistency-and-handoff.md)
- [ADR-0049: Transaction metadata treatment](0049-transaction-metadata-treatment.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Follow-up register](../follow-ups.md) (Q-007, Q-009, Q-123)
