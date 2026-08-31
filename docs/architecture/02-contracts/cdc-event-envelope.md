---
type: Architecture Review Topic
title: CDC event envelope
description: Defines the Kafka message contract required for deterministic projection updates.
tags: [contracts, kafka, cdc, events]
sources:
  - resource: ../../design/system.md
    title: System design draft
  - resource: ../../assets/example-cdc-enveloppe.json
    title: Provided Debezium MongoDB event example
status: accepted
decision_id: ADR-0004
---

# CDC event envelope

## Decision

Specify how the engine consumes the existing upstream Debezium envelope without
requiring the upstream team to publish a new canonical contract.

## Accepted boundary

Treat the Debezium MongoDB envelope as an external contract and consume it as
published. The engine validates and interprets the fields it needs rather than
rewriting the Kafka record or requiring a new upstream envelope.

The current example demonstrates a top-level `before`, `after`, `source`, `op`,
`ts_ms`, `transaction`, and `updateDescription`. It also demonstrates that
`after` is a JSON-encoded string, `updateDescription.updatedFields` is another
JSON-encoded string, `before` may be null for an update, and several source
metadata values are strings or nullable fields.

The engine-level contract for this decision is therefore:

- Decode a valid Debezium record and recognize the connector/event shape.
- Support only explicitly handled operation codes; an unsupported operation is
  unprocessable.
- Resolve the entity ID from the Kafka key or the configured document image
  (`before`/`after`) according to the manifest; inability to resolve it is
  unprocessable.
- Parse the document images and update description according to their published
  representation, including JSON-encoded strings.
- Retain source metadata needed for diagnostics and later ordering decisions,
  without assuming that `source.ts_ms`, `source.ord`, or top-level `ts_ms` is a
  globally comparable revision.
- Send malformed, unsupported, or otherwise unprocessable records to the
  durable DLQ in the engine-owned MongoDB metadata store. Advance the Kafka
  offset only after DLQ custody succeeds.
- Defer a size limit decision, but emit a bounded, actionable diagnostic signal
  whenever the engine cannot safely decode, buffer, or process an oversized
  record.

Kafka keying remains source-entity identity where provided; projection
convergence must not rely on cross-topic order.

## Example-based observations

These are observations from [the provided example](../../assets/example-cdc-enveloppe.json),
not guarantees for every Debezium version or connector configuration:

- The record is an update (`op: "u"`) for MongoDB database `side-test` and
  collection `tasks`.
- `after` is present as a serialized full document while `updateDescription`
  carries a serialized changed-field map.
- `before` is null, so the engine cannot require a pre-image for updates.
- The example has no standalone event ID; Kafka coordinates and source metadata
  may be needed for diagnostics and deduplication decisions.
- `source.snapshot` is the string `"false"`, not a JSON boolean.

## Pros

- Avoids requiring upstream changes to an existing Debezium deployment.
- Preserves the original record for replay and DLQ diagnostics.
- Makes connector-specific quirks explicit at one engine boundary.
- Lets later decisions determine which Debezium metadata is safe for ordering.

## Cons and risks

- The engine is coupled to Debezium's published shape and configuration.
- JSON-encoded nested strings add parsing and size overhead.
- The example does not prove a globally comparable revision or standalone event
  ID exists.
- Debezium version/configuration changes may alter available images and fields.

## Questions to stamp

- Which Debezium operation codes and snapshot modes are supported in the first
  release?
- What is the fallback priority among Kafka key, `after._id`, and `before._id`
  when resolving entity identity?
- What metric, log event, and diagnostic fields identify a record that exceeds a
  later size limit or cannot be safely buffered?
- How should transaction metadata be treated when present?
