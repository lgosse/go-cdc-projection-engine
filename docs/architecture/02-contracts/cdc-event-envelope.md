---
type: Architecture Review Topic
title: CDC event envelope
description: Defines the Kafka message contract required for deterministic projection updates.
tags: [contracts, kafka, cdc, events]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: in-review
---

# CDC event envelope

## Decision to stamp

Specify the event fields and upstream guarantees the engine requires.

## Draft proposal

Require a versioned envelope containing event ID, source service/database/
collection, operation, canonical entity ID, source revision or ordering token,
event time, capture time, full post-image or declared patch semantics, deletion
metadata, schema version, and trace context. Kafka keying remains source-entity
identity; projection convergence must not rely on cross-topic order.

## Pros

- Makes idempotency, replay, and diagnostics possible.
- Explicit post-image versus patch semantics prevent ambiguous merges.
- Decouples the engine from a particular CDC connector payload.

## Cons and risks

- Upstream producers may not provide a reliable revision or full post-image.
- Envelope evolution requires coordinated compatibility rules.
- Source-key partitioning provides no order for the assembled projection.

## Questions to stamp

- Who owns envelope normalization and schema registration?
- Are transactions or multi-document changes represented?
- How are oversized records and unavailable post-images handled?
