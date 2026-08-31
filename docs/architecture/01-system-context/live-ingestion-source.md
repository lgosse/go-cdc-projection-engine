---
type: Architecture Decision
title: Live ingestion source
description: Establishes Kafka as the projection engine's sole live CDC input.
tags: [context, kafka, cdc, ingestion, accepted]
sources:
  - resource: ../../design/system.md
    title: System design draft
  - resource: ../../design/otel.md
    title: OpenTelemetry design draft
status: accepted
decision_id: ADR-0001
---

# Live ingestion source

## Decision

Kafka is the projection engine's only live input. MongoDB remains the business
source of truth and is used by bootstrap, reconciliation, and repair modes.
MongoDB change-stream capture may exist upstream, but this engine does not
consume MongoDB change streams or maintain their resume tokens.

## Boundaries and non-goals

- The upstream CDC producer owns change-stream capture, normalization, and
  publication into Kafka.
- This decision does not select the CDC connector, Kafka schema technology,
  retention period, or checkpoint store.
- This decision does not resolve Redis durability, cache-miss policy,
  transformation execution, or offset advancement semantics.

## Rationale

One live input gives the engine one event contract, one checkpoint model, and one
replay path. It also makes the Kafka-based scaling and ordering assumptions in
the system draft authoritative while keeping source capture outside the generic
projection runtime.

## Alternative rejected

Consuming MongoDB change streams directly, or supporting Kafka and MongoDB change
streams as interchangeable live inputs, would duplicate ingestion and recovery
semantics and make event ordering and progress ambiguous.

## Consequences

- The Kafka event envelope becomes a blocking upstream contract.
- Kafka retention and MongoDB rebuild capability jointly define recovery limits.
- Change-stream metrics and resume-token telemetry belong to the upstream CDC
  producer unless later introduced as an explicitly separate integration.
- Direct foreign-database reads remain a separate cache/runtime decision.

## Validation evidence

- Document the Kafka envelope with identity, operation, revision/order data,
  deletion information, and replay metadata.
- Demonstrate duplicate delivery, replay, and partition-contiguous recovery in
  integration tests.
- Document recovery after Kafka retention expiry using MongoDB rebuilds.
- Assign or remove the change-stream telemetry from the engine observability
  model.

## Review trigger

Revisit if Kafka cannot provide the required CDC completeness or retention,
MongoDB rebuilds cannot meet recovery objectives, or a second live source becomes
an explicit product requirement.

## Related concepts

- [Source draft conflicts](../source-conflicts.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Data-store responsibilities](data-store-responsibilities.md)
