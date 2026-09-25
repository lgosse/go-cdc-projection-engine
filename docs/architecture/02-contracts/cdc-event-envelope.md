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
- Apply only Debezium operation codes `c` (create), `u` (update), and `d`
  (delete) in v1. Operation `r` (snapshot read) and unknown operation codes
  are unprocessable; they do not enter the projection apply path.
- If `source.snapshot` is present, accept only boolean `false` or string
  `"false"`. Any other present value is unsupported and unprocessable. The
  field may be absent on a live event.
- Do not consume Debezium startup data-snapshot rows in v1. Populate existing
  data through the engine's separate bootstrap scan and CDC handoff in
  [ADR-0015](../decisions/0015-bootstrap-consistency-and-handoff.md). This
  remains true even if a connector emits an `r` event after configuration or
  runtime changes.
- The v1 connector profile must suppress startup data-snapshot rows. The
  checked-in Terraform configuration currently declares
  `snapshot.mode: never`; a connector mode that emits startup `r` rows is
  outside the supported profile. The engine still DLQs any `r` it receives.
- Resolve identity according to [ADR-0005](../decisions/0005-identity-and-ordering-scope.md):
  for create/update events, use the configured ID from `after` (MongoDB
  `_id` in the example); for deletes, use the configured ID from `before`.
  Fall back to the Kafka key only when that relevant image has no ID.
- If the relevant image and Kafka key both provide IDs and they disagree, the
  event is unprocessable. If neither provides an ID, the event is also
  unprocessable. In either case, use the durable DLQ custody rule above.
- Parse the document images and update description according to their published
  representation, including JSON-encoded strings.
- Treat the top-level `transaction` field as optional diagnostic context:
  absent, `null`, and object values are accepted; a non-null non-object value is
  unprocessable and uses durable DLQ custody. Keep transaction metadata and
  `source.lsid`/`source.txnNumber` out of v1 freshness fencing and ordering. Do
  not wait for transaction-boundary or sibling events, and do not promise
  transaction-level atomicity across documents or target stores. See
  [ADR-0049](../decisions/0049-transaction-metadata-treatment.md).
- Retain source metadata needed for diagnostics and later ordering decisions,
  without assuming that `source.ts_ms`, `source.ord`, or top-level `ts_ms` is a
  globally comparable revision.
- Send malformed, unsupported, or otherwise unprocessable records to the
  durable DLQ in the engine-owned MongoDB metadata store. Advance the Kafka
  offset only after DLQ custody succeeds.
- Keep the numeric per-event size limit open until capacity evidence sets it.
  Once configured, a record that exceeds that limit or cannot be safely decoded
  is an event-local deterministic failure and goes to the durable DLQ before
  its Kafka offset can complete.
- Distinguish that event-local failure from temporary worker-queue saturation:
  pause Kafka intake and report queue pressure while the queue is full; do not
  DLQ a valid event solely because the queue is temporarily busy. A record that
  cannot fit even when the bounded queue has capacity is an event-local failure.
- Identify rejected records with counters for failure and DLQ-custody outcomes,
  a record-size histogram, and structured diagnostics containing a stable error
  code, failure scope, operation, outcome, applicable record/limit byte counts,
  and trace/span correlation. Keep metric dimensions bounded. Put source
  coordinates and the original event or protected payload reference in durable
  DLQ/protected diagnostics, not ordinary telemetry; telemetry export never
  blocks processing.

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
- Top-level `transaction`, `source.lsid`, and `source.txnNumber` are null in
  this example; that is an observed sample, not a requirement that all events
  lack transaction metadata.

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

- **Resolved by [ADR-0047](../decisions/0047-debezium-operation-and-snapshot-scope.md):**
  Which Debezium operation codes and snapshot modes are supported in the first
  release?
- **Resolved by [ADR-0005](../decisions/0005-identity-and-ordering-scope.md):**
  What is the fallback priority among Kafka key, `after._id`, and `before._id`
  when resolving entity identity?
- **Resolved by [ADR-0048](../decisions/0048-event-size-and-buffer-diagnostics.md):**
  What metric, log event, and diagnostic fields identify a record that exceeds
  a later size limit or cannot be safely buffered?
- **Resolved by [ADR-0049](../decisions/0049-transaction-metadata-treatment.md):**
  How should transaction metadata be treated when present?
