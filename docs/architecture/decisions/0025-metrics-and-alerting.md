---
type: Architecture Decision Record
title: "ADR-0025: Metrics and alerting"
description: Canonical metrics cover progress, saturation, dependencies, correctness, and lifecycle; symptom-based alerts use bounded dimensions and owned runbooks.
tags: [architecture, adr, observability, metrics, alerting, slo]
status: accepted
decision_id: ADR-0025
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Canonical metrics cover freshness/progress, throughput/saturation, dependencies/sinks, correctness/custody, and lifecycle.
  - Metrics use only bounded dimensions from ADR-0024; entity identifiers, raw keys, payloads, and arbitrary error text are excluded.
  - Alerts are symptom-based, use sustained windows, and have an owner, action, and runbook; isolated expected events do not page.
  - Alerting uses the initial freshness and recovery objectives from ADR-0020, with exact windows and thresholds remaining operational follow-ups.
  - Telemetry and backend outages never block source processing or offset decisions.
---

# ADR-0025: Metrics and alerting

## Context

The draft defines useful metrics but mixes namespaces, fixed thresholds, and
unbounded identifiers. Approved performance and telemetry decisions require
freshness and recovery objectives, bounded dimensions, privacy-safe diagnostics,
and non-blocking telemetry export.

## Decision

Maintain one canonical metric surface organized into five groups:

1. **Freshness and progress:** source-watermark age, end-to-end processing
   latency, Kafka lag, last successful progress, paused partitions, and blocked
   work.
2. **Throughput and saturation:** received events, effective mutations, queue
   records/bytes/age, batch size/age, CPU, memory, and dependency saturation.
3. **Dependency and sink behavior:** Elasticsearch bulk latency, failures and
   throttling; Redis hit/miss and lookup latency; source-read latency/rate;
   Kafka reconnects; and rebalance activity.
4. **Correctness and custody:** source-fence rejections, deterministic failures,
   DLQ writes and custody failures, reconciliation discrepancies and repairs, and
   migration verification failures.
5. **Lifecycle:** bootstrap chunks, checkpoints, migration phases, and cache
   generation rebuild progress.

Use counters for totals, gauges for current state, and histograms for latency or
size distributions. Derive rates and ratios from counters over time. Use bounded
dimensions from ADR-0024—projection, version, mode, operation, outcome,
dependency, and stable error code. Topic and partition labels are limited to
progress metrics only when their cardinality is explicitly bounded. Never use
entity IDs, cache keys, raw payloads, or arbitrary error text.

Alert on sustained user-visible symptoms and stuck progress. Page for DLQ
custody failure, metadata/fence risk, freshness SLO breach, stalled work,
active schema mismatch, failed migration verification, or sink failure that
threatens retention/recovery objectives. Create warnings or tickets for rising
lag, retries, throttling, cache misses, drift, slow bootstrap/repair, or frequent
rebalances. Keep expected fence drops, normal volume, and isolated poison events
dashboard-only unless their rate becomes abnormal.

Every page or ticket has an owner, explicit action, and runbook. Use sustained
windows and rates, and provide maintenance/migration suppression or severity
adjustment. Telemetry export or backend failure never blocks processing.

## Alternatives considered

1. Expose every draft metric and alert on fixed values. This is easy initially
   but creates noisy alerts, high cardinality, and thresholds unrelated to
   workload mode or approved objectives.
2. Alert on any single DLQ event or fence drop. This treats expected event-local
   behavior as an incident and causes alert fatigue.
3. Make telemetry export part of processing success. This provides stronger
   signal completeness but lets an observability outage block source progress.

## Consequences

- Metric names, instrument types, allowed dimensions, and error codes become
  maintained operational contracts.
- Alert rules require owners, runbooks, maintenance handling, and periodic
  tuning.
- Metrics remain bounded while detailed entity diagnosis uses protected traces,
  diagnostic views, or DLQ references.
- A telemetry backend outage may lose signals but cannot corrupt or pause data
  processing.

## Validation

- Every approved freshness, recovery, and correctness objective has a metric and
  actionable alert.
- Alerts identify the affected projection/workload and recommended action.
- No metric family grows with entity count.
- Normal fence drops and isolated poison events do not create alert storms.
- DLQ custody, stalled progress, and migration failures page reliably.
- Exporter/backend outages do not block processing or offset advancement.

## Review triggers

Revisit if alerts miss incidents, page too noisily, metric cardinality exceeds
budget, ownership/runbooks become stale, or approved objectives change.

## Related concepts

- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Telemetry conventions](../06-observability/telemetry-conventions.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
