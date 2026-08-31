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
- For create/update events, prefer the configured ID from `after`; fall back to
  the Kafka key when the image does not contain it.
- For delete events, prefer the configured ID from `before`; fall back to the
  Kafka key.
- If the Kafka key and document image both provide an ID and disagree, the event
  is unprocessable and goes to DLQ.
- Use a source-scoped Debezium ordering tuple or an entity revision for freshness
  fencing only after the available connector fields and scope are documented.
- Store fences per contributing entity, not only at the root projection.
- Do not compare timestamps alone across topics, collections, replica sets, or
  independent Debezium sources.
- Do not claim a total order across related entities. Kafka offsets describe
  delivery progress only.
- An event missing required identity is unprocessable. An event lacking usable
  ordering metadata must not be applied with a timestamp- or offset-based
  last-write-wins rule; it is deferred or repaired under a later explicit policy.

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
- Canonical ID serialization may differ from existing consumer expectations.

## Questions to stamp

- Which exact Debezium source fields are available in every production connector
  configuration, and what is their comparison scope?
- What canonical serialization is required for ObjectID, UUID, string, and
  numeric IDs?
- Are equal source revisions possible, and what deterministic tie-breaker is
  available?
- Should snapshot-read events participate in normal fencing or use a separate
  bootstrap path?
