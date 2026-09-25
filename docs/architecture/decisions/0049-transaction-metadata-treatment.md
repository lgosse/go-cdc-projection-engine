---
type: Architecture Decision Record
title: "ADR-0049: Transaction metadata treatment"
description: Defines the v1 treatment of Debezium transaction metadata in change events.
tags: [architecture, adr, cdc, debezium, transactions, contracts]
status: accepted
decision_id: ADR-0049
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Absent, null, or object-valued top-level transaction metadata is accepted; a non-null non-object value is unprocessable.
  - Transaction metadata and MongoDB session/transaction source fields are diagnostic context only in v1 and do not define event freshness or atomic grouping.
  - V1 does not wait for transaction boundaries or related change events before processing an event.
---

# ADR-0049: Transaction metadata treatment

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

The supplied MongoDB Debezium event has `transaction: null`, with `source.lsid`
and `source.txnNumber` also null. Transaction metadata may be available in
other connector configurations, but the v1 event contract must remain safe
without it. The engine already uses per-entity source fencing; Kafka offsets
are delivery checkpoints, not a global source revision ([ADR-0005](0005-identity-and-ordering-scope.md)).

Treating transaction metadata as an atomicity contract would require the engine
to correlate multiple change events and transaction-boundary events, buffer
them durably, and define recovery and partial-write behavior across projection
documents and target stores. Transaction identifiers and within-transaction
positions alone do not supply those behaviors.

## Decision

- Treat the top-level `transaction` field as optional advisory metadata.
  Absence and `null` are valid. An object is accepted and retained as opaque
  event metadata; the engine does not require or interpret fields such as
  `id`, `total_order`, or `data_collection_order`.
- A non-null `transaction` value that is not an object is an unprocessable
  envelope and follows the durable DLQ custody rule in
  [ADR-0012](0012-backpressure-retry-dlq-policy.md) and [ADR-0048](0048-event-size-and-buffer-diagnostics.md).
- Treat `source.lsid` and `source.txnNumber` as optional opaque diagnostic
  context. Do not use them, or fields inside `transaction`, as v1 freshness
  fences, ordering keys, or substitutes for the source-scoped fields still
  under review in Q-007.
- Process each accepted change event using the existing identity, fencing, and
  delivery rules. Do not wait for sibling events or transaction-boundary
  records, and do not promise source-transaction atomicity across entities,
  projection documents, or target stores.
- Do not copy transaction metadata into projected documents or routine
  telemetry. It may remain available as protected event context for
  diagnostics; original-event retention follows the existing DLQ and privacy
  rules.

## Alternatives considered

1. **Buffer and group changes by source transaction.** This could preserve
   transaction membership and within-transaction order, but requires
   transaction-boundary consumption, durable buffers, crash recovery, limits
   for large or incomplete transactions, and explicit behavior for partial
   writes across targets. It is outside v1.
2. **Discard transaction metadata entirely.** This avoids retaining unused
   fields, but removes potentially useful diagnostic context and makes a later
   transaction-aware extension harder to investigate.

## Consequences

- Events with or without transaction metadata follow one v1 apply path and do
  not block waiting for other collections or records.
- A projection may briefly reflect only part of a multi-document source
  transaction. V1 makes no cross-document atomic-visibility guarantee.
- Transaction metadata cannot be mistaken for a globally comparable event
  revision. Any future transaction-aware behavior requires a superseding
  decision and explicit evidence for buffering, recovery, and target writes.

## Validation

- Events with `transaction` absent, `null`, or an object are accepted when
  their other required fields are valid; a non-null scalar or array reaches
  durable DLQ custody before its offset completes.
- An object-valued transaction field does not delay application while the
  engine waits for related events or boundary records.
- V1 freshness and ordering decisions do not use transaction fields,
  `source.lsid`, or `source.txnNumber`.
- Projected documents and routine metric/log fields do not expose transaction
  metadata.

## Review triggers

Revisit before promising transaction-level visibility or consistency, before
using transaction metadata for ordering, or when a connector/plugin change
causes the supported envelope shape or metadata requirements to change.

## Related concepts

- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Backpressure, retry, and DLQ](0012-backpressure-retry-dlq-policy.md)
- [Follow-up register](../follow-ups.md) (Q-004 and Q-007)
