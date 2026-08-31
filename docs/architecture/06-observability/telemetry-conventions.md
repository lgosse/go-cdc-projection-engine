---
type: Architecture Review Topic
title: Telemetry conventions
description: Defines OpenTelemetry identity, attribute ownership, and propagation rules.
tags: [observability, opentelemetry, conventions]
sources:
  - resource: ../../design/otel.md
    title: OpenTelemetry design draft
status: proposed
---

# Telemetry conventions

## Decision to stamp

Define resource attributes, instrumentation scope, semantic conventions, and
cardinality/privacy budgets shared across signals.

## Draft proposal

Use standard OpenTelemetry resource and messaging/database semantic conventions
where applicable. Put stable process identity such as service and deployment on
the resource; put projection name, mode, and manifest version on measurements or
spans only when bounded. Propagate upstream trace context from Kafka and create
new linked traces for batch/coalesced work when parentage is ambiguous.

## Pros

- Produces interoperable telemetry across backends.
- Separates stable resource identity from per-workload dimensions.
- Trace links model batch causality better than arbitrary parent selection.

## Cons and risks

- Projection dimensions multiply time-series count across many manifests.
- Upstream trace context may be missing or untrusted.
- Semantic conventions and backend support evolve over time.

## Questions to stamp

- Which attributes are mandatory on every signal?
- What cardinality and retention budgets apply?
- Which collector and backends are platform responsibilities?
