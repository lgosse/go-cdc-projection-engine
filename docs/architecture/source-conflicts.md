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

The live-ingestion portion of this proposal is accepted in
[ADR-0001](decisions/0001-live-ingestion-source.md). The remaining datastore,
cache, transformation, and offset conflicts stay open.

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
- **Resolved by [Redis authority boundary](01-system-context/redis-authority-boundary.md):**
  Redis is non-authoritative; required projector state must be durable or
  reconstructible elsewhere. The exact durable owner remains open.
- Transformations described as Go/Bloblang execution while a follow-up topic
  implies Bloblang-to-Painless translation.
- Manual batch offset commits versus committing "remaining" records after
  partial bulk failures without defining partition-contiguous progress.

## Questions to stamp

- Which durable component owns each checkpoint and control-plane state?
- What must happen when an upstream event is incomplete or malformed?
- What is the cache-miss policy when the engine cannot query a foreign database?
