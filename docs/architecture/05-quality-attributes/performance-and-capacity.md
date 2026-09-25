---
type: Architecture Review Topic
title: Performance and capacity
description: Defines measurable throughput, latency, document, and fan-out limits.
tags: [quality, performance, capacity]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0020
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Live freshness targets are initially p95 <= 30 seconds and p99 <= 2 minutes for accepted events under normal load.
  - A one-hour outage must be recoverable within two hours after dependencies recover, subject to benchmark validation.
  - Hard safety limits protect queues, memory, documents, nested items, fan-out, batches, source reads, and effective mutations; performance targets generate alerts rather than silent drops.
  - Sustainable capacity is planned at approximately two times peak effective mutation rate until representative benchmarks establish a better margin.
  - Unsafe or unmeasured manifests are rejected or blocked before production use; deterministic oversized records reach durable DLQ custody before offset completion, while temporary queue saturation pauses intake ([ADR-0048](../decisions/0048-event-size-and-buffer-diagnostics.md)).
---

# Performance and capacity

## Decision

Define capacity as a measured, per-workload and per-manifest envelope rather than
one global throughput number. Live streaming, bootstrap, replay, and
reconciliation/repair have separate objectives and budgets.

The initial live freshness objective is p95 (95th percentile) of accepted events
becoming searchable within 30 seconds and p99 (99th percentile) within 2 minutes
under normal load. Bootstrap has no fixed elapsed-time contract: it must finish
before CDC retention and recovery safety margins are exhausted. As an initial
recovery target, a one-hour outage should be caught up within two hours after
dependencies recover. Repair is lower priority than live ingestion and has an
independent rate limit.

Hard safety limits apply to queue records/bytes/age, event and canonical
document size, nested-child count, relationship fan-out, batch bytes/age,
memory per partition, controlled source-read rate, transformation complexity
and evaluation work, and effective mutations per event. Exceeding a hard limit
pauses, rejects, defers, or otherwise isolates the affected work under the
existing delivery and DLQ rules. Throughput, lag,
utilization, transformation latency, and approaching limits are operational
warnings and scaling signals, not reasons to silently drop data.

Capacity is measured in effective mutations:

```text
Kafka events x relationship fan-out x number of write targets
```

Until benchmarks establish a better margin, plan sustainable capacity at roughly
two times peak effective mutation rate. Manifest-level limits describe one
projection; deployment configuration governs shared worker, datastore, and
repair capacity. Representative multi-store benchmarks are a gate for
production manifests and must include peak, high-fan-out, hot-key, cache-miss,
dual-write, duplicate, out-of-order, and dependency-outage scenarios.

Transformation benchmarks also record mapping compile complexity, Kafka event
bytes, canonical input and output bytes, relation and array cardinalities,
transform CPU/latency, and peak worker memory under concurrency. Numeric caps
for these dimensions are recorded with their workload profile and evidence under
Q-070 ([ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md)).

Oversized Debezium records emit bounded metrics and structured diagnostics.
When an incoming record exceeds its configured per-event limit or cannot be
safely decoded, it reaches durable DLQ custody before its offset completes.
Temporary queue saturation pauses intake instead of creating a DLQ record for a
valid event. The numeric per-event limit remains subject to capacity evidence.
See [ADR-0048](../decisions/0048-event-size-and-buffer-diagnostics.md).

## Pros

- Prevents arbitrary constants such as 1,000 events or 500 milliseconds becoming
  accidental contracts.
- Makes unsafe manifests rejectable before production.
- Supports evidence-based worker grouping and scaling.
- Accounts for fan-out and dual-write cost rather than treating every event as
  equivalent.

## Cons and risks

- Representative multi-store benchmarks are costly to maintain.
- A single global limit may waste capacity or reject valid workloads.
- P99 targets depend heavily on managed dependency behavior.
- Conservative hard limits may require large relations to become separate
  projections.

## Alternatives considered

1. Use one global set of batch, worker, and throughput limits. This is simple
   but either rejects safe workloads or permits unsafe high-fan-out workloads.
2. Rely on benchmarks without declared hard limits. This detects problems late
   and allows unsafe manifests into production.
3. Scale only on Kafka event rate. This ignores fan-out, dual-write targets,
   document size, and dependency-specific bottlenecks.

## Consequences

- Capacity planning and observability must report effective mutations, not only
  Kafka records.
- Manifest validation and deployment configuration each own different classes
  of limits.
- Benchmark datasets and scenarios become maintained engineering evidence.
- Recovery capacity must include enough headroom to catch up without starving
  live traffic.
- Large or high-cardinality relations may need separate projections instead of
  larger nested limits.

## Validation

- Unsafe document, nested, fan-out, memory, source-read, and batch configurations
  are rejected or bounded before production.
- Live freshness and recovery targets hold under representative load and
  dual-write migration conditions.
- Hot keys and high fan-out cannot starve unrelated work.
- Queue pressure pauses intake before process capacity is exhausted.
- Oversized records are diagnosable and repair/terminal handling is auditable.

## Review trigger

Revisit when benchmark results contradict the initial targets, workload shape or
fan-out changes materially, recovery objectives change, or limits threaten
capacity or search freshness.

## Related concepts

- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)

## Follow-up questions

- What benchmark datasets and projected workload distributions should be kept
  current?
- What exact hard limits and warning thresholds replace the initial targets?
- **Resolved by [ADR-0056](../decisions/0056-nested-relation-capacity-boundaries.md):** When must a high-cardinality nested relation become a separate projection? When it exceeds the tested hard envelope for cardinality, document size, or required performance; numeric values remain evidence-gated by Q-070.
- Q-070 also establishes the numeric, per-relation live fan-out ceilings for reference propagation before production use ([ADR-0058](../decisions/0058-reference-fanout-execution-threshold.md)).
- Q-070 also records numeric Bloblang complexity, byte, collection-work, latency, and worker-resource limits with representative workload evidence ([ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md)).
- Do observed workloads justify changing the two-times peak planning margin?
