---
type: Architecture Decision Record
title: "ADR-0024: Telemetry conventions"
description: OpenTelemetry uses stable resource identity, bounded dimensions, protected diagnostics, sampled traces, and non-blocking collector export.
tags: [architecture, adr, observability, opentelemetry, telemetry, privacy]
status: accepted
decision_id: ADR-0024
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Stable service identity is carried on telemetry resources; workload and operation attributes are scope-appropriate and bounded.
  - Metrics never use entity IDs, cache keys, raw source identifiers, or arbitrary error text as dimensions.
  - Upstream Kafka trace context is optional and untrusted; tracing never affects correctness or source progress.
  - The engine exports to a platform-owned OpenTelemetry Collector through OTLP; exporter failure is bounded and non-blocking.
  - Raw payloads, secrets, and sensitive values are excluded from normal telemetry; protected diagnostics may use controlled pseudonymous references.
---

# ADR-0024: Telemetry conventions

## Context

The draft defines telemetry for streaming and batch modes but mixes resource and
workload attributes, includes high-cardinality identifiers, and proposes raw
payloads in error logs. Observability must remain useful across projections
without becoming a privacy leak or a processing dependency.

## Decision

Use OpenTelemetry conventions with a separation between stable resource identity
and bounded operation context. Every signal carries service name, engine version,
and deployment environment. Workload signals may carry projection name, version,
mode, source system, operation, outcome, and bounded dependency attributes.
Signals without projection context are not forced to invent one.

Metrics must use only bounded dimensions. They must not use document IDs, Kafka
keys, cache keys, raw Mongo identifiers, arbitrary error messages, or unbounded
relationship names as labels. Protected diagnostic traces and logs may include a
pseudonymous entity reference or source coordinate when needed, but never raw
payloads or sensitive values.

Accept upstream Kafka trace context only after size and format validation. Create
a new processing span when context is absent. Use trace links for coalesced
batches when there is no single meaningful parent. Trace propagation and
telemetry export never affect processing correctness, offset advancement, or
backpressure.

Retain ordinary event traces at a low baseline sample rate, while retaining
failures, slow operations, SLO violations, migrations, bootstrap chunks, repairs,
and operator actions. The collector may apply tail-based retention when
available. Batch and control-plane spans remain observable even when individual
events are sampled.

Use one canonical metric namespace aligned with freshness, throughput,
saturation, correctness, migration, reconciliation, and DLQ custody. Share
stable error codes across logs, metrics, and DLQ records. Emit structured JSON
logs with trace correlation and bounded diagnostics.

Export through OTLP to a platform-owned OpenTelemetry Collector. Collector
routing, backend selection, authentication, and retention are platform
responsibilities. Local buffers and retries are bounded; exporter failure emits a
bounded internal signal and never pauses Kafka processing.

## Alternatives considered

1. Instrument separately for each metrics, tracing, and logging backend. This
   exposes backend-specific features but creates multiple naming systems and
   migration paths.
2. Trace every event and attach every identifier. This improves ad hoc debugging
   but is expensive, high-cardinality, and unsafe for sensitive data.
3. Make telemetry export part of processing success. This gives stronger signal
   completeness but allows an observability outage to block source progress.

## Consequences

- Telemetry schemas, error codes, and allowed dimensions become maintained
  contracts.
- Metrics remain queryable at scale, while detailed entity diagnosis moves to
  sampled protected traces or DLQ references.
- The platform must operate a compatible collector and telemetry backends.
- Exporter outages may lose some telemetry but cannot corrupt or pause data
  processing.
- Incident sampling and retention need operational controls separate from normal
  event sampling.

## Validation

- Metric cardinality remains bounded as projections and entities grow.
- Raw payloads, secrets, and sensitive values do not appear in normal telemetry.
- Traces correlate intake, coalescing, transformation, writes, and repairs when
  sampled.
- Exporter outages do not block Kafka progress or unsafe offset decisions.
- Shared error codes correlate metrics, logs, traces, and DLQ records.
- Protected diagnostics and DLQ references are sufficient to investigate rare
  failures.

## Review triggers

Revisit if telemetry cardinality or retention exceeds platform budgets, if the
collector cannot preserve required context, if sampling misses incidents, or if
privacy policy changes permitted diagnostic attributes.

## Related concepts

- [Telemetry conventions](../06-observability/telemetry-conventions.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
