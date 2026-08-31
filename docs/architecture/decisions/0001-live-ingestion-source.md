---
type: Architecture Decision Record
title: "ADR-0001: Live ingestion source"
description: Kafka is the projection engine's sole live CDC input.
tags: [architecture, adr, kafka, cdc, ingestion]
status: accepted
decision_id: ADR-0001
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0001: Live ingestion source

## Context

The system draft specifies Kafka topics, consumer groups, manual offsets, and a
Kafka-based streaming pipeline. The observability draft also describes MongoDB
change-stream reconnects and resume-token checkpoints. Supporting both inside
the projection engine would create competing live-input, checkpoint, replay, and
ordering semantics.

## Decision

Kafka is the projection engine's only live input. MongoDB remains the business
source of truth and is used by bootstrap, reconciliation, and repair modes.
MongoDB change-stream capture may exist upstream, but this engine does not
consume MongoDB change streams or maintain their resume tokens.

## Boundaries

- The upstream CDC producer owns change-stream capture, normalization, and Kafka
  publication.
- This decision does not select the connector, Kafka schema technology,
  retention period, or checkpoint store.
- This decision does not resolve cache-miss, transformation, Redis durability,
  or offset advancement policy.

## Alternatives considered

1. Consume MongoDB change streams directly. This reduces dependence on an
   upstream Kafka CDC producer, but couples the engine to source capture and
   duplicates recovery logic.
2. Support Kafka and MongoDB change streams as interchangeable live inputs. This
   offers deployment flexibility, but creates ambiguous ordering, duplicate
   delivery, and checkpoint ownership when both paths are present.

## Consequences

- The Kafka event envelope is a blocking upstream contract.
- Kafka retention and MongoDB rebuild capability jointly define recovery limits.
- Change-stream metrics and resume-token telemetry belong upstream unless a
  separate integration is explicitly accepted.
- The engine's stream-mode scaling and ordering model can be designed around one
  live source.

## Validation

- Document an envelope with identity, operation, revision/order data, deletion
  information, and replay metadata.
- Test duplicate delivery, replay, and partition-contiguous recovery.
- Document recovery after Kafka retention expiry using MongoDB rebuilds.
- Assign or remove change-stream telemetry from the engine observability model.

## Review trigger

Revisit if Kafka cannot provide required CDC completeness or retention, MongoDB
rebuilds cannot meet recovery objectives, or a second live source becomes an
explicit product requirement.

## Related concepts

- [Live ingestion source](../01-system-context/live-ingestion-source.md)
- [Source draft conflicts](../source-conflicts.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
