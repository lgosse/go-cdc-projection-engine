---
type: Architecture Review Topic
title: Metrics and alerting
description: Defines service-level metrics and symptom-based actionable alerts.
tags: [observability, metrics, alerts, slo]
sources:
  - resource: ../../design/system.md
    title: System design draft
  - resource: ../../design/otel.md
    title: OpenTelemetry design draft
status: proposed
---

# Metrics and alerting

## Decision to stamp

Choose a small canonical metric surface aligned with freshness, correctness,
throughput, saturation, and failure custody.

## Draft proposal

Measure source-record receipt, partition lag, end-to-end event age, processing
and bulk latency, queue saturation, retry outcomes, cache availability, fencing,
DLQ custody, bootstrap progress, migration phase, and reconciliation drift.
Alert on sustained user-visible symptoms and stuck progress, with thresholds
derived from SLOs rather than fixed draft numbers.

## Pros

- Supports incident response across every operating mode.
- Progress and event-age signals reveal stalls that request counts miss.
- Canonical names avoid the two incompatible metric namespaces in the drafts.

## Cons and risks

- Item, projection, topic, and error labels can create high cardinality.
- Counters cannot directly express ratios without reliable denominators.
- Drift and DLQ alerts require clear operator ownership to remain useful.

## Questions to stamp

- What are the freshness and correctness SLOs?
- Which dimensions are permitted on each metric?
- Which alerts page, ticket, or only inform dashboards?
