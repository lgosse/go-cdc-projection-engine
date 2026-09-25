---
type: Architecture Review Topic
title: Identity, time, and ordering
description: Defines canonical keys and deterministic stale-event comparison.
tags: [contracts, identity, ordering, correctness]
sources:
  - resource: ../../design/system.md
    title: System design draft
  - resource: ../../assets/example-cdc-enveloppe.json
    title: Provided Debezium MongoDB event example
  - resource: https://debezium.io/documentation/reference/2.7/connectors/mongodb.html
    title: Debezium MongoDB connector documentation
status: accepted
decision_id: ADR-0005
---

# Identity, time, and ordering

## Decision

Use manifest-defined source identity and source-scoped freshness fencing. Kafka
partition/offset coordinates are delivery checkpoints only; they are not a
global event revision.

## Accepted direction

- The manifest defines the canonical source ID field and serialization.
- Canonical IDs use a type tag and length-framed payload. ObjectID uses 12 raw
  bytes; UUID uses 16 RFC/network-order bytes (legacy BSON subtype 3 requires
  explicit byte-order configuration); strings preserve exact UTF-8 bytes;
  signed BSON `int32` and `int64` remain distinct. Unsupported or mismatched
  types fail closed ([ADR-0053](../decisions/0053-canonical-source-id-serialization.md)).
- For create/update events, prefer the configured ID from `after`; fall back to
  the Kafka key when the image does not contain it.
- For delete events, prefer the configured ID from `before`; fall back to the
  Kafka key.
- Debezium snapshot-read (`op: "r"`) events are not applied through the live
  stream path in v1; they are unprocessable and use durable DLQ custody as
  defined by [ADR-0047](../decisions/0047-debezium-operation-and-snapshot-scope.md).
- If the Kafka key and document image both provide an ID and disagree, the event
  is unprocessable and goes to DLQ.
- For a MongoDB source, require `source.ts_ms` and `source.ord` for freshness,
  namespaced by `source.name` and `source.rs`; compare them only for the same
  database, collection, and canonical entity ID. A larger timestamp wins, with
  `ord` breaking ties. Do not compare across source names or replica sets
  ([ADR-0052](../decisions/0052-mongodb-source-ordering-scope.md)).
- For the same scoped entity and revision, an identical source mutation is a
  duplicate no-op. Different source mutations at that revision are an integrity
  conflict: preserve current state, durably retain the conflict, and hold that
  entity for authoritative repair without an arrival-time tie-breaker
  ([ADR-0054](../decisions/0054-equal-source-revision-conflicts.md)).
- `source.wallTime` remains the bootstrap-boundary field under ADR-0015, not a
  per-entity freshness revision. The outer event `ts_ms` is connector
  processing time and does not fence source state.
- Under [ADR-0049](../decisions/0049-transaction-metadata-treatment.md),
  transaction metadata and `source.lsid`/`source.txnNumber` are diagnostic
  context only in v1; they are not freshness fences or ordering keys.
- Store fences per contributing entity, not only at the root projection.
- Do not compare timestamps alone across topics, collections, replica sets, or
  independent Debezium sources.
- Do not claim a total order across related entities. Kafka offsets describe
  delivery progress only.
- An event missing required identity is unprocessable. An event lacking usable
  ordering metadata must not be applied with a timestamp- or offset-based
  last-write-wins rule; it is deferred or repaired under a later explicit policy.
- Production source enablement requires verifying the deployed Debezium plugin
  version and representative `c`/`u`/`d` events contain the required fields
  (Q-123). Missing or incomparable source scope fails closed.

## Pros

- Avoids treating arrival order as source freshness.
- Handles identity from the actual Debezium image/key shapes.
- Makes per-entity fencing explicit for replay and cross-topic assembly.
- Prevents false total-order guarantees across independent sources.

## Cons and risks

- A usable source-scoped revision may not be available in every connector
  configuration.
- Per-entity metadata increases projection size and script complexity.
- Deferred events without ordering metadata need durable custody and repair.
- Existing consumer expectations or stored fences may use a different ID
  serialization and require a rebuild or migration.

## Questions to stamp

- **Resolved by [ADR-0052](../decisions/0052-mongodb-source-ordering-scope.md):** Which exact Debezium source fields are available in every production connector configuration, and what is their comparison scope? The required tuple is `source.ts_ms` + `source.ord`, scoped by source name, replica set, database, collection, and entity ID; production field verification remains an enablement gate tracked by Q-123.
- **Resolved by [ADR-0053](../decisions/0053-canonical-source-id-serialization.md):** What canonical serialization is required for ObjectID, UUID, string, and numeric IDs? Use type-tagged, length-framed payloads; preserve exact string bytes; distinguish signed BSON integer widths; reject unsupported or mismatched kinds.
- **Resolved by [ADR-0054](../decisions/0054-equal-source-revision-conflicts.md):** Are equal source revisions possible, and what deterministic tie-breaker is available?
  The source contract expects different mutations to have different revisions.
  Identical redelivery is a no-op; differing mutations at one revision are
  quarantined for authoritative repair, with no arrival, offset,
  processing-time, or payload-order tie-break.
- What production deployment and event evidence verifies the Debezium plugin
  version and required source fields for create, update, and delete events?
- **Deferred by [ADR-0047](../decisions/0047-debezium-operation-and-snapshot-scope.md):**
  Should snapshot-read events participate in normal fencing or use a separate
  bootstrap path? Revisit before enabling any connector or workflow that can
  emit `r` events into an engine input stream.
