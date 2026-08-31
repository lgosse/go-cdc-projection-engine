---
type: Architecture Decision Record
title: "ADR-0026: Tracing and logging"
description: Tracing follows meaningful processing boundaries, ordinary event spans are sampled, and structured logs use protected diagnostics without raw payloads.
tags: [architecture, adr, observability, tracing, logging, privacy]
status: accepted
decision_id: ADR-0026
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Traces cover meaningful event, batch, sink-flush, bootstrap, repair, migration, and operator boundaries.
  - Ordinary event traces use a low baseline sample rate; failures, slow operations, objective violations, and control-plane work are retained.
  - Structured logs use stable error codes and trace correlation while excluding raw payloads, secrets, Restricted fields, and unprotected identifiers.
  - Protected diagnostics may expose controlled pseudonymous references and source coordinates to authorized operators.
  - Telemetry export remains bounded and non-blocking; DLQ and control-plane records provide durable failure context.
---

# ADR-0026: Tracing and logging

## Context

The design draft proposes trace trees and structured logs for streaming and
batch modes, but includes raw payloads and document identifiers in normal log
examples. At the engine's event volume, tracing every event in full would also
create excessive cost and cardinality. ADR-0024 and ADR-0025 require bounded,
privacy-safe, non-blocking telemetry.

## Decision

Create traces for meaningful units of work: sampled event-processing spans,
batch/coalescing spans, Elasticsearch bulk flushes, bootstrap chunks,
reconciliation/repair operations, migrations, and operator actions. For a
coalesced batch with multiple possible parents, use trace links rather than an
arbitrary parent. Tracing never affects processing, offset advancement, retry,
or backpressure decisions.

Use a low baseline sample rate for ordinary event traces, initially around 1%.
Retain all errors and DLQ routing, slow operations, freshness or recovery
objective violations, bootstrap chunks, migrations, repairs, and operator
actions. Incident-mode sampling may be increased through configuration. Where
available, the collector may apply tail-based retention. Batch and control-plane
spans remain observable even when individual events are sampled.

Emit structured JSON logs containing timestamp, severity, service/version,
environment, projection/version/mode, operation, stable error code, outcome,
failure scope, trace/span correlation, and safe source or DLQ/repair references.
Do not log raw CDC envelopes, credentials, Restricted fields, cache keys, or
unprotected document identifiers. Replace raw payload fields such as the draft's
`raw_payload` error field with a protected DLQ or diagnostic reference.

Use the same stable error taxonomy as metrics and DLQ records. Protected
diagnostic traces and logs may include a pseudonymous entity reference and source
coordinates when needed by an authorized operator, but never raw payloads or
sensitive values.

Telemetry export uses bounded buffers and retries through the platform collector.
Exporter or backend failure may lose telemetry and emits a bounded health signal,
but never blocks Kafka processing or offset decisions. Durable DLQ and
engine-metadata records remain the source for retained failure context.

## Alternatives considered

1. Trace every event and include every identifier in logs. This eases ad hoc
   debugging but creates high cost, high-cardinality telemetry, and privacy
   exposure.
2. Log only aggregate metrics. This is cheap but makes rare event failures,
   migration issues, and repair outcomes difficult to investigate.
3. Make telemetry export part of processing success. This improves signal
   completeness but allows an observability outage to block source progress.

## Consequences

- Trace boundaries follow actual processing and control-plane work rather than
  every internal function.
- Detailed entity diagnosis requires protected traces, diagnostic views, or DLQ
  references instead of ordinary logs.
- Sampling and redaction reduce cost and exposure but need incident controls and
  operator access governance.
- Exporter outages may lose telemetry but cannot corrupt or pause data
  processing.

## Validation

- A sampled event can be correlated through intake, coalescing, transformation,
  and writes.
- Failures, slow operations, and control-plane actions are retained reliably.
- Raw payloads, secrets, Restricted fields, and unprotected identifiers do not
  appear in ordinary telemetry.
- DLQ and protected diagnostic references are sufficient to investigate rare
  failures.
- Collector/backend outages do not block processing or offset advancement.
- Batch, bootstrap, migration, and repair traces remain useful under load.

## Review triggers

Revisit if sampling misses incidents, trace/log retention exceeds budget,
diagnostic indirection slows recovery, or privacy policy changes permitted
telemetry attributes.

## Related concepts

- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Telemetry conventions](../06-observability/telemetry-conventions.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
