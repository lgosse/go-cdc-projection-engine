---
type: Architecture Review Topic
title: Performance and capacity
description: Defines measurable throughput, latency, document, and fan-out limits.
tags: [quality, performance, capacity]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Performance and capacity

## Decision to stamp

Define the workload envelope that batching, nested documents, Redis lookups, and
Elasticsearch scripts must support.

## Draft proposal

Set targets per workload class for events per second, end-to-end freshness,
bootstrap duration, bulk size and age, projection document bytes, nested item
count, transformation time, relation fan-out, and memory per partition. Derive
defaults only after representative benchmarks.

## Pros

- Prevents arbitrary constants such as 1,000 events or 500 milliseconds becoming
  accidental contracts.
- Makes unsafe manifests rejectable before production.
- Supports evidence-based worker grouping and scaling.

## Cons and risks

- Representative multi-store benchmarks are costly to maintain.
- A single global limit may waste capacity or reject valid workloads.
- P99 targets depend heavily on managed dependency behavior.

## Questions to stamp

- What current and projected workload distributions matter?
- Which limits are hard validation errors versus warnings?
- How much catch-up capacity is required after an outage?
