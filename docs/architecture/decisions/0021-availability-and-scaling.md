---
type: Architecture Decision Record
title: "ADR-0021: Availability and scaling"
description: Search, ingestion, and recovery availability are separated; workload-group consumers degrade by scope and scale on sustained multi-signal pressure.
tags: [architecture, adr, availability, scaling, kafka, kubernetes]
status: accepted
decision_id: ADR-0021
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Search-read health, ingestion readiness, and recovery/backlog state are reported separately.
  - Production stream workload groups start with at least two replicas, and scaling is bounded by useful partitions and tested dependency capacity.
  - Consumer groups are organized by shared failure and capacity domains; strict isolation may use separate groups, while shared groups retain partition-coupled progress.
  - Scaling uses sustained lag/freshness plus saturation signals with stabilization and cooldown periods.
  - Dependency failures pause or block the smallest safe scope after bounded retry; ordinary outages do not cause crash loops.
---

# ADR-0021: Availability and scaling

## Context

Search reads, Kafka progress, and recovery have different failure and
freshness characteristics. A projector outage need not make an existing
Elasticsearch target unreadable, while a dependency outage can make it unsafe to
advance offsets. Replica count and Kafka consumer topology also affect whether a
projection failure is isolated or coupled to unrelated work.

## Decision

Separate availability into search-read, ingestion-progress, and recovery
dimensions. Elasticsearch's active read target should remain queryable when
projector workers are unavailable, while freshness degradation is visible. Kafka
progress advances only for work whose dependencies and terminal-custody
requirements are healthy. Paused work must resume within the performance and
recovery objectives and before retention safety margins expire.

Use cooperative-sticky Kafka assignment and bounded graceful draining during
shutdown and rebalance. Start with at least two stream replicas per production
workload group. Do not scale beyond the useful partition count or tested sink
capacity. Organize consumer groups by shared failure and capacity domains;
separate groups are appropriate where strict projection isolation justifies
duplicate Kafka reads. When projections share a group and partition, a blocked
projection may hold that partition's committed progress; the system must not
claim independent progress in that topology.

Scale on sustained combined signals: Kafka lag, source-watermark freshness,
queue utilization and age, Elasticsearch bulk latency/throttling, Redis or
source-read latency/rate, CPU, memory, and rebalance frequency. Stabilization and
cooldown windows prevent oscillation. Lag alone is insufficient because adding
consumers against a saturated dependency can reduce throughput.

Degrade at the smallest safe scope: malformed manifests or mappings block a
projection; event-local failures use the durable DLQ; partition-specific issues
pause a partition; shared dependency outages block the dependent workload; and
corrupted process invariants may terminate the process. Ordinary dependency
outages use bounded retry and scoped pause rather than crash loops. Readiness
exposes process/liveness, search-read, ingestion, and degraded-projection state
separately.

## Alternatives considered

1. Use one shared consumer group and scale globally. This is efficient but lets
   a blocked projection or hot workload couple unrelated progress.
2. Use one consumer group per projection. This maximizes isolation but multiplies
   Kafka reads, coordination, and resource cost.
3. Couple search readiness to projector health. This simplifies health reporting
   but makes search unavailable during ingestion outages even when the active
   target remains valid.

## Consequences

- Search may remain available while freshness and ingestion readiness degrade.
- Consumer-group topology becomes a capacity and failure-domain decision, not
  merely a deployment detail.
- At least two production replicas and graceful draining add baseline resource
  cost.
- Health and alerting must distinguish search, ingestion, dependency, and
  recovery states.
- Scaling requires multi-signal metrics and stabilization rather than a single
  lag threshold.

## Validation

- Search remains queryable while ingestion is paused or projector workers are
  restarted.
- A broken projection does not corrupt or incorrectly advance unrelated work.
- Graceful rebalance preserves contiguous offset semantics.
- Scaling improves sustained effective throughput without oscillation or
  dependency overload.
- Dependency outages do not create crash loops or DLQ floods.
- Recovery resumes within the performance and disaster-recovery objectives.

## Review triggers

Revisit if search freshness or ingestion objectives are missed, consumer-group
coupling blocks recovery, replica cost exceeds capacity, or dependency failure
scopes cannot be isolated safely.

## Related concepts

- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
