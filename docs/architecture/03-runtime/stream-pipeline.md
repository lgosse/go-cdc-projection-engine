---
type: Architecture Review Topic
title: Stream pipeline
description: Defines deterministic live-event processing and coalescing boundaries.
tags: [runtime, streaming, batching, transformations]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: in-review
---

# Stream pipeline

## Decision to stamp

Define the stages and intermediate representation from Kafka record to projected
document mutation.

## Draft proposal

Decode and validate events, resolve affected root IDs, batch required lookups,
group mutations by projection document, order or reduce them using entity fences,
evaluate transformations against a canonical intermediate document, and emit one
idempotent mutation per target document and active physical index.

## Pros

- Coalescing reduces write amplification.
- A canonical intermediate form supports shared semantics across modes.
- Per-document mutation boundaries match Elasticsearch atomicity.

## Cons and risks

- Coalescing records from multiple Kafka partitions complicates commits.
- A partial event stream may not reconstruct the canonical document in memory.
- Large hot documents can dominate a batch and create skew.

## Questions to stamp

- What state is required to assemble transformations during stream processing?
- What are the maximum batch age, size, and memory budget?
- Can coalescing cross topic or partition boundaries safely?
