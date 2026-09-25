---
type: Architecture Decision Record
title: "ADR-0048: Event size and buffer diagnostics"
description: Defines how oversized records and temporary queue saturation are handled and observed.
tags: [architecture, adr, kafka, capacity, observability, dlq]
status: accepted
decision_id: ADR-0048
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - A deterministic per-record size or decode failure is stored in the durable DLQ before its Kafka offset completes.
  - Temporary worker-queue saturation pauses intake and does not create a DLQ record for a valid event.
  - Routine telemetry uses bounded dimensions and excludes entity keys, document identifiers, and raw payloads.
  - The numeric per-event size limit remains open until capacity evidence establishes it.
---

# ADR-0048: Event size and buffer diagnostics

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

Q-003 asks how operators can identify records that exceed a future size limit
or cannot be safely buffered. The numeric event limit still needs representative
capacity evidence. The engine nevertheless needs a safe terminal rule for an
individual bad record and a distinct response to temporary queue pressure.
Existing delivery rules require durable DLQ custody before an offset can
complete ([ADR-0012](0012-backpressure-retry-dlq-policy.md)).

## Decision

- Do not set a numeric per-event size limit in this decision. Capacity work
  establishes that limit before a production manifest relies on it.
- Treat an event that exceeds the configured per-event limit or cannot be safely
  decoded as a deterministic, event-local failure. Persist the event in the
  durable DLQ before completing its Kafka offset.
- Treat a temporarily full worker queue as backpressure. Pause Kafka intake
  while the queue is full; do not DLQ an otherwise valid event just because
  workers are temporarily busy. If an event cannot fit even when the bounded
  queue has capacity because the event itself exceeds its limit, use the
  event-local DLQ rule.
- Record failure and DLQ-custody outcomes with counters, record size with a
  histogram, and queue pressure with bounded queue size/age and pause-state
  measurements. Use only bounded labels such as projection, version, mode,
  operation, outcome, dependency, and stable error code.
- Structured diagnostics may include stable error code, failure scope,
  operation, outcome, applicable record and configured-limit byte counts, and
  trace/span correlation. Keep source coordinates and the original event or a
  protected payload reference in durable DLQ/protected diagnostics, not routine
  logs or metric labels. A telemetry-export failure does not block processing.

## Alternatives considered

1. **Put payloads and source keys in routine logs.** This makes individual
   incidents easy to search, but exposes event content and creates unbounded
   telemetry cardinality.
2. **Use aggregate metrics only.** This limits data exposure, but makes an
   individual bad record difficult to investigate. Protected DLQ diagnostics
   retain the event-level evidence needed for authorized investigation.
3. **DLQ every event received while a queue is full.** This is simple to
   implement, but misclassifies valid work as bad input and can turn a temporary
   throughput dip into a burst of durable failure records.

## Consequences

- A deterministic invalid or oversized event has a durable terminal outcome;
  its offset cannot complete before that outcome is stored.
- Queue saturation is visible as a pause/backpressure condition and resolves
  through resumed intake when capacity returns.
- Operators can track rate, size, and queue pressure through bounded telemetry
  and investigate individual records through protected DLQ diagnostics.
- Capacity evidence must still select the numeric event limit and verify that
  the bounded queue, DLQ, and telemetry path behave safely at that limit.

## Validation

- An oversized or undecodable record reaches durable DLQ custody before its
  Kafka offset completes.
- A valid event held behind temporary queue saturation is not sent to the DLQ;
  intake pauses and resumes as queue capacity changes.
- A record that cannot fit even into an otherwise available bounded queue is
  classified as an event-local failure.
- Metric labels remain bounded and routine telemetry contains no document ID,
  Kafka key, or raw payload.
- Telemetry export failure does not prevent event processing or offset handling.

## Review triggers

Revisit when capacity evidence sets the numeric per-event limit, the queue or
DLQ durability model changes, or operator investigations show that the
protected diagnostic path is insufficient.

## Related concepts

- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Backpressure, retry, and DLQ](0012-backpressure-retry-dlq-policy.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Telemetry conventions](../06-observability/telemetry-conventions.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Follow-up register](../follow-ups.md) (Q-003)
