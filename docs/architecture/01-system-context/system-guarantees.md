---
type: Architecture Review Topic
title: System guarantees
description: Defines the externally meaningful correctness and availability promises.
tags: [context, guarantees, semantics]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# System guarantees

## Decision to stamp

Turn aspirational phrases such as zero downtime and order agnosticism into
bounded, testable guarantees.

## Draft proposal

Promise at-least-once event handling, deterministic convergence for supported
event histories, no cross-service MongoDB point reads in stream mode, rebuildable
Elasticsearch projections, and read availability during compatible migrations.
Define explicit exceptions for dependency outages, retention loss, malformed
events, and unsupported schema changes.

## Pros

- Creates verifiable service-level semantics.
- Avoids implying exactly-once behavior from manual Kafka commits.
- Forces assumptions about time, deletion, and replay into the contract.

## Cons and risks

- Deterministic convergence is difficult for partial updates and derived fields.
- At-least-once delivery moves idempotency complexity into every sink operation.
- "Zero downtime" still needs a measurable read/write availability target.

## Questions to stamp

- What freshness and recovery objectives are promised?
- Is convergence required after arbitrary reorderings or only within a retention
  window?
- Which guarantees apply separately to stream, bootstrap, migration, and repair?
