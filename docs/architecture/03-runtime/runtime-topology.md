---
type: Architecture Review Topic
title: Runtime topology
description: Defines process isolation, consumer grouping, and work ownership.
tags: [runtime, topology, kafka, concurrency]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Runtime topology

## Decision to stamp

Choose how projections, topics, partitions, and worker pools are assigned across
processes and pods.

## Draft proposal

Run one binary with explicit modes, but isolate stream workloads into a small
number of projection groups with independent consumer-group identities,
concurrency limits, and failure domains. Group only workloads with compatible
throughput and latency profiles; do not force all manifests into one consumer
group.

## Pros

- Limits rebalance and poison-manifest blast radius.
- Allows independent scaling of hot and ordinary projections.
- Retains one deployable engine.

## Cons and risks

- More groups increase deployment and configuration complexity.
- A source topic feeding several projections may be consumed multiple times.
- Poor grouping can still create noisy-neighbor behavior.

## Questions to stamp

- Is deployment per projection, per source domain, or per workload class?
- What unit owns a Kafka partition at runtime?
- How are resource budgets enforced between manifests in one process?
