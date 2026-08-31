---
type: Architecture Decision Record
title: "ADR-0004: Debezium envelope boundary"
description: The engine consumes the existing Debezium envelope as published and DLQs unprocessable records.
tags: [architecture, adr, kafka, debezium, cdc, dlq]
status: accepted
decision_id: ADR-0004
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0004: Debezium envelope boundary

## Context

The upstream Kafka CDC envelope is produced by an existing Debezium deployment
outside the projection engine team's control. The provided example contains
connector-specific details such as JSON-encoded `after` and
`updateDescription.updatedFields` values, nullable `before`, string-valued
snapshot metadata, and no standalone event ID.

## Decision

Consume the existing Debezium envelope as published. The engine may parse and
normalize it internally for processing, but it must not require an upstream
envelope redesign or rewrite the Kafka record. The original record must remain
available for diagnostics and replay.

If a record cannot be decoded, has an unsupported operation or event shape,
cannot resolve the configured entity identity, or cannot be safely applied, the
engine classifies it as unprocessable and writes it to the durable DLQ in the
engine-owned MongoDB metadata store before advancing its Kafka offset.

Oversized-record thresholds are deferred. If size prevents safe decoding,
buffering, or processing, the engine must emit a bounded, actionable diagnostic
signal.

Transient dependency failures are not unprocessable data; they remain subject to
retry and defer policy.

## Boundaries

- Debezium operation-code support, identity precedence, transaction handling, and
  size thresholds are follow-up decisions.
- This ADR does not make Debezium metadata a globally comparable ordering
  revision or require a standalone event ID.
- DLQ custody and offset advancement follow [ADR-0003](0003-durable-state-ownership.md)
  and the later offset-semantics decision.

## Alternatives considered

1. Require upstream to publish a new canonical envelope. This is outside the
   engine's authority and would delay integration with the existing stream.
2. Accept connector-specific payloads opportunistically without a defined
   unprocessable path. This risks silent data loss and inconsistent replay.

## Consequences

- The engine is coupled to the deployed Debezium shape and configuration.
- Internal parsing must handle serialized nested values and nullable metadata.
- The original Kafka value and source coordinates become important DLQ evidence.
- Upstream Debezium changes require compatibility review even without an engine
  code change.

## Validation

- Cover the provided example with a parser and interpretation fixture.
- Verify original-envelope preservation in DLQ records or durable references.
- Verify DLQ custody precedes Kafka offset advancement.
- Exercise malformed, unsupported, missing-identity, and size-diagnostic paths.
- Later stamp operation, identity, transaction, and size matrices explicitly.

## Review trigger

Revisit if the upstream Debezium version/configuration changes, if a new event
shape is introduced, or if DLQ/replay requirements cannot preserve the original
record and source coordinates.

## Related concepts

- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Live ingestion source](../01-system-context/live-ingestion-source.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
