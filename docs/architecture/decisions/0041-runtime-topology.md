---
type: Architecture Decision Record
title: "ADR-0041: Runtime topology"
description: Stream workload groups use compatible capacity and failure domains with bounded per-projection work and partition-contiguous completion.
tags: [architecture, adr, runtime, topology, kafka, concurrency]
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

# ADR-0041: Runtime topology

## Context

The engine can subscribe multiple projections to shared Kafka topics. A single
consumer group is efficient but couples partition progress and failure scope;
one group per projection isolates failures but duplicates Kafka reads. Accepted
offset, backpressure, availability, manifest, and deployment decisions require
bounded queues, independent projection outcomes, and graceful partition handoff.

## Decision

Organize stream workloads into groups with compatible capacity and failure
domains. Each group uses one stream executable, one Kafka consumer group, one
Kubernetes Deployment with multiple replicas, a compatible manifest bundle, and
bounded per-partition and per-projection queues and budgets.

Start with one stream workload group for the first representative projection.
Add projections to an existing group only when freshness, fan-out, dependency,
retry, and failure characteristics are compatible. Use separate consumer groups
when strict failure isolation, materially different scaling, or incompatible
retention/retry behavior justifies duplicate Kafka reads.

Dispatch each source event to every subscribed projection in its group. Each
projection has an independent validation and terminal outcome. Shared Kafka
offset progress advances only after all required projection outcomes complete.

Kafka owns partition assignment. The assigned consumer owns partition intake and
in-flight work until graceful handoff. Internal worker pools may process records
concurrently, but per-partition completion tracking preserves contiguous commit
semantics. A revoked partition is drained within the bounded deadline and its
previous owner cannot commit after revocation.

Within each pod, bound queues by records, bytes, and age; enforce per-projection
concurrency and rate budgets; preserve fairness; and expose explicit pause or
block state. No unbounded queue is shared by all manifests, and a hot or blocked
projection cannot silently starve compatible neighbors.

## Alternatives considered

1. **One consumer group per projection.** Maximizes isolation but duplicates
   Kafka reads and multiplies deployment and capacity costs.
2. **One global consumer group.** Efficient but lets a blocked or hot projection
   hold shared partition progress.
3. **Group only by source topic.** Mixes workloads with incompatible capacity and
   failure profiles.
4. **One Deployment per projection by default.** Clear but unnecessarily
   expensive for compatible projections.
5. **One pod-level pool without per-projection budgets.** Creates starvation and
   noisy-neighbor risks.

## Consequences

- Group membership is a capacity and failure-domain decision.
- Per-projection budgets, status, and outcome tracking are required within a
  shared group.
- Consumer-group topology and duplication cost belong in capacity evidence.
- Graceful rebalance and partition ownership are tested runtime contracts.
- Projections can be isolated by moving them to separate groups without making
  Redis or Elasticsearch authoritative.

## Validation

- A source event updates all subscribed projections with independent outcomes.
- Shared offsets do not advance until every required projection outcome is
  terminal.
- A blocked or hot projection cannot starve compatible neighbors.
- Rebalances and pod termination preserve partition-contiguous completion.
- Separate groups isolate failure and scaling when criteria are met.
- Kafka duplication and resource cost remain within capacity objectives.

## Review triggers

Revisit if shared groups repeatedly couple unrelated progress, group membership
cannot be classified from workload evidence, or duplicate Kafka reads threaten
capacity objectives.

## Related concepts

- [Runtime topology](../03-runtime/runtime-topology.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
