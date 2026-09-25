---
type: Architecture Decision Record
title: "ADR-0045: Fallback-read observability and privacy"
description: Defines how bounded source-of-truth fallback reads are observed without exposing protected identifiers.
tags: [architecture, adr, observability, privacy, cache]
status: accepted
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Fallback metrics use bounded dimensions and stable outcome/error values.
  - Ordinary telemetry excludes raw payloads, entity identifiers, cache keys, and unprotected source coordinates.
  - Authenticated diagnostics may use controlled pseudonymous references when individual-event investigation requires them.
  - Telemetry export remains bounded and non-blocking for processing.
---

# ADR-0045: Fallback-read observability and privacy

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

ADR-0043 permits a source-of-truth read only through an explicit, bounded
relation policy. The source OpenTelemetry draft proposed logging lookup keys and
document identifiers. Accepted telemetry decisions instead require bounded
metric dimensions, exclude raw identifiers from ordinary telemetry, and reserve
controlled pseudonymous references for protected diagnostics. Follow-up Q-121
asks how fallback attempts and failures should be made observable within those
boundaries.

## Decision

Observe each source fallback with bounded metrics for attempt count, outcome,
and latency. Use the accepted low-cardinality dimensions: projection, version,
mode, operation, outcome, dependency, and stable error code. Represent fallback
as a bounded operation value and outcomes as a stable finite set. Do not attach
entity IDs, source keys, cache keys, raw payloads, arbitrary relationship names,
or arbitrary error text to metric dimensions.

Create a source-fallback span for permitted lookups. Structured logs and spans
carry the operation, outcome, stable error code, duration, and trace/span
correlation. Ordinary logs and traces omit missing keys and unprotected entity
identifiers. Retain failures and slow operations under the accepted sampling
policy. Telemetry export failure must not block processing or offset progress.

The authenticated, bounded diagnostic path may include a controlled
pseudonymous entity reference or source coordinate when needed to investigate
an individual fallback. It never exposes raw payloads, cache keys, or
unprotected entity identifiers. This ADR does not choose the pseudonymization
mechanism, retention period, or exact operator authorization; those remain
governed by their respective follow-ups.

## Alternatives considered

1. **Log raw lookup keys for direct debugging.** This makes individual misses
   easy to search but exposes identifiers and encourages high-cardinality
   telemetry, contrary to the accepted security and telemetry boundaries.
2. **Keep only aggregate metrics and omit diagnostic references.** This further
   reduces linkability and implementation work, but makes rare fallback
   failures slower to investigate. The accepted protected-diagnostics boundary
   provides a controlled investigation path.

## Consequences

- Operators can identify fallback volume, latency, source failures, and outcome
  patterns without indexing metrics by entity.
- Event-level investigation uses trace correlation and, when required,
  authenticated diagnostics with a controlled pseudonymous reference.
- The diagnostic pseudonymization mechanism, retention, and access details must
  follow the existing security and observability follow-ups.

## Validation

- Fallback attempts and outcomes produce bounded counters and latency samples
  using only approved dimensions.
- Varying entity IDs and cache keys does not increase metric cardinality.
- Ordinary logs and traces contain no raw payload, cache key, missing key, or
  unprotected entity identifier.
- Authorized diagnostics can correlate a protected reference to an investigation
  without returning raw payloads or unprotected identifiers.
- Collector or backend failure does not block source progress.

## Review triggers

Revisit if operators cannot diagnose fallback incidents with the approved
signals, metric cardinality exceeds its budget, privacy requirements change, or
the protected diagnostic path proves insufficient.

## Related concepts

- [Source draft conflicts](../source-conflicts.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Telemetry conventions](../06-observability/telemetry-conventions.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
