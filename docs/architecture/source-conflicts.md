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
status: proposed
---

# Source draft conflicts

## Decision to stamp

Resolve the draft's internal contradictions before treating any detailed
pipeline behavior as authoritative.

## Draft proposal

Treat Kafka as the only live event source and MongoDB as the bootstrap,
reconciliation, and repair source of truth. Treat Elasticsearch as a disposable
read-model store, Redis as operational state/cache only, and remove direct
MongoDB change-stream and runtime fallback assumptions unless later accepted.

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

- Kafka topics in the system draft versus MongoDB change streams and resume
  tokens in the telemetry draft.
- "Zero cross-database point queries at runtime" versus cache-miss database
  fallbacks in the telemetry draft.
- Redis described as non-persistent cache while also owning migration routing
  state that must survive loss.
- Transformations described as Go/Bloblang execution while a follow-up topic
  implies Bloblang-to-Painless translation.
- Manual batch offset commits versus committing "remaining" records after
  partial bulk failures without defining partition-contiguous progress.

## Questions to stamp

- Which statements above are intentional requirements and which are draft
  residue?
- Which component owns each durable checkpoint and control-plane state?
- What must happen when an upstream event is incomplete or malformed?
