---
type: Architecture Review Topic
title: Source draft conflicts
description: Lists incompatible assumptions in the source drafts that require explicit decisions.
tags: [architecture, conflicts, ingestion, correctness]
sources:
  - resource: ../design/system.md
    title: System design draft
  - resource: ../design/otel.md
    title: OpenTelemetry design draft
status: in-review
---

# Source draft conflicts

## Decision to stamp

Resolve the draft's internal contradictions before treating any detailed
pipeline behavior as authoritative.

## Draft proposal

The live-ingestion, durable-state, and Debezium-boundary portions of this
proposal are accepted in [ADR-0001](decisions/0001-live-ingestion-source.md),
[ADR-0003](decisions/0003-durable-state-ownership.md), and
[ADR-0004](decisions/0004-debezium-envelope-boundary.md). Cache-miss,
transformation, and offset-semantics concerns are tracked by their accepted ADRs
or remaining follow-up topics. The transformation execution boundary is accepted;
its exact subset, resource limits, and context lifecycle remain open. Operation
support, identity fallback, transaction handling, and size diagnostics remain
explicit follow-up topics under the accepted Debezium boundary.

## Pros

- Matches the stated Kafka-first system boundary.
- Gives each datastore a clear role and failure contract.
- Avoids building two competing checkpoint and ingestion models.

## Cons and risks

- Requires the Kafka CDC producer and its event contract to be trustworthy.
- A strict no-fallback policy makes cache completeness a readiness concern.
- Some telemetry proposed for change streams becomes irrelevant or must move to
  the upstream CDC producer.

## Conflicts to resolve

- **Resolved by [live ingestion source](01-system-context/live-ingestion-source.md):**
  Kafka topics are the engine input; MongoDB change-stream capture and resume
  tokens belong upstream.
- "Zero cross-database point queries at runtime" versus cache-miss database
  fallbacks in the telemetry draft.
- **Resolved by [Redis authority boundary](01-system-context/redis-authority-boundary.md)
  and [durable state ownership](01-system-context/durable-state-ownership.md):**
  Redis is non-authoritative; Kafka owns consumer cursors and the engine-owned
  MongoDB metadata store owns migration, control-plane, and DLQ state.
- **Resolved by [Debezium envelope boundary](02-contracts/cdc-event-envelope.md):**
  the existing upstream envelope is consumed as published; unprocessable events
  go to the durable DLQ before offset advancement.
- **Resolved in principle by [identity and ordering scope](02-contracts/identity-time-ordering.md):**
  Kafka offsets track delivery only; source-scoped metadata fences freshness. The
  exact tuple and comparison scope remain a follow-up decision.
- **Resolved by [transformation execution boundary](02-contracts/transformation-contract.md):**
  Bloblang-in-Go is authoritative; Painless performs only fenced mutations.
- Manual batch offset commits versus committing "remaining" records after
  partial bulk failures without defining partition-contiguous progress remains
  an offset-semantics question; offsets are not freshness revisions.

## Questions to stamp

- What is the cache-miss policy when the engine cannot query a foreign database?
