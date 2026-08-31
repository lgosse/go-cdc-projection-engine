---
type: Architecture Review Topic
title: Tracing and logging
description: Defines trace boundaries, sampling, structured events, and sensitive-data controls.
tags: [observability, tracing, logging, privacy]
sources:
  - resource: ../../design/otel.md
    title: OpenTelemetry design draft
status: proposed
---

# Tracing and logging

## Decision to stamp

Define useful trace units and structured log events without recording every
entity ID or raw payload by default.

## Draft proposal

Trace batches, flushes, bootstrap chunks, and control-plane operations; use
sampled event-level spans only for targeted diagnostics. Emit structured lifecycle
and failure logs with trace correlation, stable error codes, source coordinates,
and manifest identity. Keep raw events and high-cardinality document IDs out of
normal logs; expose them through access-controlled diagnostic workflows.

## Pros

- Controls telemetry cost at high event rates.
- Retains correlation for slow or failed operations.
- Reduces privacy and secret leakage risk.

## Cons and risks

- Sampling can miss rare record-specific failures.
- Batch spans obscure individual-event latency.
- Diagnostic indirection slows ad hoc debugging.

## Questions to stamp

- What sampling policy changes during incidents?
- Which stable error taxonomy is shared by logs, metrics, and DLQ?
- What payload fields, if any, may appear in operator telemetry?
