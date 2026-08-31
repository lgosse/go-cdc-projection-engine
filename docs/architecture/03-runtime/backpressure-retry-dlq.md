---
type: Architecture Review Topic
title: Backpressure, retry, and DLQ
description: Defines failure classification, overload control, and poison-event custody.
tags: [runtime, retry, backpressure, dlq]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Backpressure, retry, and DLQ

## Decision to stamp

Define bounded queues, retry budgets, terminal error classification, and replay
custody without turning pod crashes into the default recovery mechanism.

## Draft proposal

Apply bounded per-workload buffers and pause Kafka consumption before memory is
exhausted. Retry transient failures with jittered exponential backoff and an
elapsed-time budget. Send deterministic data/configuration failures to a durable
DLQ envelope containing source coordinates, raw payload reference, manifest
version, target outcomes, and diagnostics. Reserve process termination for
corrupted invariants or unrecoverable process state.

## Pros

- Prevents retry storms and uncontrolled memory growth.
- Preserves enough context for safe replay.
- Distinguishes dependency incidents from bad records.

## Cons and risks

- Error classification can be wrong as dependencies evolve.
- Raw payload retention may expose sensitive data.
- A durable DLQ still needs ownership, retention, and replay controls.

## Questions to stamp

- What are queue and retry budgets per mode?
- Where does the DLQ live and who owns remediation?
- Which errors block a projection rather than isolate one record?
