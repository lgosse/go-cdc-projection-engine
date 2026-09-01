---
type: Architecture Review Topic
title: Runtime topology
description: Defines process isolation, consumer grouping, and work ownership.
tags: [runtime, topology, kafka, concurrency]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0041
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Stream workload groups are organized by compatible capacity and failure domains rather than one global or one-per-projection default.
  - Each group uses one stream executable, one Kafka consumer group, and a Kubernetes Deployment with bounded per-partition and per-projection work.
  - Kafka owns partition assignment; internal concurrency preserves partition-contiguous completion and bounded graceful handoff.
  - One event may update all subscribed projections with independent terminal outcomes before shared offset progress advances.
  - Separate consumer groups are used when strict failure isolation or materially different workload behavior justifies duplicate Kafka reads.
---

# Runtime topology

## Decision

Organize stream workloads into groups with compatible capacity and failure
domains. Each group has one stream executable, one Kafka consumer group, one
Kubernetes Deployment with multiple replicas, a compatible manifest bundle, and
bounded per-partition and per-projection queues and budgets.

Start with one stream workload group for the first representative projection.
Add projections to an existing group only when freshness, fan-out, dependency,
retry, and failure characteristics are compatible. Use separate consumer groups
when strict failure isolation, materially different scaling, or incompatible
retention/retry behavior justifies duplicate Kafka reads.

Dispatch a source event to every subscribed projection in the group. Each
projection has independent validation and terminal outcome. Shared Kafka offset
progress advances only after all required projection outcomes complete under the
accepted delivery rules.

Kafka owns partition assignment. The consumer assigned to a partition owns its
intake and in-flight work until graceful handoff. Internal worker pools may
process records concurrently, but per-partition completion tracking preserves
contiguous commit semantics. A revoked partition is drained within the bounded
deadline and its previous owner cannot commit after revocation.

Within each pod, enforce bounded queues by records, bytes, and age; per-projection
concurrency and rate budgets; fairness between projections; and explicit pause or
block state for affected work. No unbounded queue is shared by all manifests, and
a hot or blocked projection cannot silently starve compatible neighbors.

## Pros

- Balances Kafka read efficiency with failure and scaling isolation.
- Keeps partition-contiguous offset semantics explicit under concurrency.
- Supports multi-projection dispatch without hiding per-projection outcomes.
- Prevents hot or blocked projections from consuming unbounded shared capacity.
- Allows strict isolation when workload differences justify duplicate reads.

## Cons and risks

- Workload grouping requires explicit compatibility criteria.
- Shared groups retain some partition and progress coupling.
- Separate groups multiply Kafka reads, deployments, and operational cost.
- Cross-partition coalescing and per-projection completion increase bookkeeping.
- Moving a projection between groups may require coordinated deployment changes.

## Alternatives considered

1. One consumer group per projection. This maximizes isolation but duplicates
   Kafka reads and multiplies deployment and capacity costs.
2. One global consumer group for all projections. This is efficient but allows a
   blocked or hot projection to hold shared partition progress.
3. Group only by source topic. This mixes workloads with incompatible capacity
   and failure profiles.
4. One Deployment per projection by default. This is clear but unnecessarily
   expensive for compatible projections.
5. One pod-level pool without per-projection budgets. This creates starvation and
   noisy-neighbor risks.

## Consequences

- Group membership is a capacity and failure-domain decision, not merely a
  deployment convenience.
- Per-projection budgets, status, and outcome tracking are required within a
  shared group.
- Consumer-group topology and duplication cost belong in capacity evidence.
- Graceful rebalance and partition ownership are tested runtime contracts.
- A projection can be isolated by moving it to a separate group without making
  Redis or Elasticsearch authoritative.

## Validation

- A source event updates all subscribed projections with independent outcomes.
- Shared offsets do not advance until every required projection outcome is
  terminal.
- A blocked or hot projection cannot starve compatible neighbors.
- Rebalances and pod termination preserve partition-contiguous completion.
- Separate groups isolate failure and scaling when their criteria are met.
- Kafka duplication and resource cost remain within capacity objectives.

## Review trigger

Revisit if shared groups repeatedly couple unrelated progress, group membership
cannot be classified from workload evidence, or duplicate Kafka reads threaten
capacity objectives.

## Related concepts

- [Runtime topology](runtime-topology.md)
- [Stream pipeline](stream-pipeline.md)
- [Offsets and delivery semantics](offsets-and-delivery.md)
- [Elasticsearch writes](elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](backpressure-retry-dlq.md)
- [Cache and reverse lookups](cache-and-reverse-lookups.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
