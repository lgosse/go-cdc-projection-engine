---
type: Architecture Decision Record
title: "ADR-0005: Identity and ordering scope"
description: Kafka offsets track delivery while source-scoped metadata fences entity freshness.
tags: [architecture, adr, identity, ordering, debezium, kafka]
status: accepted
decision_id: ADR-0005
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0005: Identity and ordering scope

## Context

The existing Debezium envelope may provide a Kafka key, document images, source
timestamps, ordinals, and connector-specific metadata, but not necessarily a
standalone event ID or one globally comparable revision. Kafka ordering is scoped
to a partition and represents delivery order, not necessarily source freshness.

## Decision

Use manifest-defined source identity and source-scoped freshness fencing. Kafka
partition/offset coordinates are delivery checkpoints only; they are not a global
event revision.

For create/update events, prefer the configured ID from `after`; fall back to the
Kafka key when the image does not contain it. For delete events, prefer the
configured ID from `before`; fall back to the Kafka key. If both sources provide
an ID and disagree, the event is unprocessable and goes to DLQ.

Use a source-scoped Debezium ordering tuple or an entity revision for freshness
fencing only after the available fields and scope are documented. Store fences
per contributing entity, not only at the root projection. Never compare
timestamps alone across independent sources or treat Kafka offsets as a global
freshness version.

An event missing required identity is unprocessable. An event lacking usable
ordering metadata must not be applied with timestamp- or offset-based
last-write-wins; it is deferred or repaired under a later explicit policy.

## Boundaries

- Exact Debezium metadata tuple, comparison scope, canonical ID serialization,
  equal-revision tie-breaking, and snapshot participation remain follow-up
  decisions.
- Cross-topic and cross-entity total ordering is not promised.
- Kafka offsets remain the durable delivery cursor under the at-least-once model;
  they do not make Kafka-to-Elasticsearch writes atomic.

## Alternatives considered

1. Use Kafka topic/partition/offset as the global event revision. This follows
   arrival order but cannot safely identify stale source state across partitions.
2. Use timestamp-only last-write-wins. This is simple but sensitive to clock,
   precision, connector, and cross-source differences.

## Consequences

- Projection updates need per-entity fence metadata or an equivalent durable
  comparison mechanism.
- Events without identity are terminal data failures; events without usable
  ordering may wait for repair rather than corrupt newer state.
- Related entities can still arrive in any cross-topic order and require
  relationship-specific convergence behavior.
- ID serialization becomes a compatibility surface across stream, bootstrap,
  repair, and migration modes.

## Validation

- Test image-only, key-only, matching, mismatched, and missing identity paths.
- Generate reordered histories within and across partitions and prove no stale
  event overwrites a newer fenced entity state.
- Inventory production Debezium fields and document their comparison scope.
- Test equal-revision handling and events without usable ordering metadata.
- Verify bootstrap, replay, and repair use the same identity and fencing rules.

## Review trigger

Revisit when a production connector configuration changes, a new source type is
added, or the available Debezium metadata cannot provide the required fencing
scope and recovery objectives.

## Related concepts

- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
