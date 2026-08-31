---
type: Architecture Review Topic
title: Offsets and delivery semantics
description: Defines partition-contiguous Kafka progress and crash recovery.
tags: [runtime, kafka, offsets, delivery]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Offsets and delivery semantics

## Decision to stamp

Define exactly when a Kafka offset may advance after batching, coalescing,
dual-writing, retrying, or dead-lettering records.

## Draft proposal

Guarantee at-least-once processing. Track completion per topic partition and
commit only the highest contiguous completed offset. A record counts as complete
only after all required projection targets succeed or a durable terminal DLQ
write succeeds. Reprocessing after a crash is expected and safe through
idempotent sink operations.

## Pros

- Correctly handles out-of-order completion and partial bulk failures.
- Makes crash behavior explicit and testable.
- Avoids claiming exactly-once semantics across Kafka and Elasticsearch.

## Cons and risks

- One blocked record holds later offsets in its partition.
- Coalescing several source records into one write complicates completion mapping.
- A MongoDB DLQ adds another non-transactional durability boundary.

## Questions to stamp

- Can records be terminally skipped, and under whose authorization?
- Is the DLQ write acknowledgment sufficient to commit the source offset?
- How are rebalance revocation and in-flight batches drained?
