---
type: Architecture Decision Record
title: "ADR-0020: Performance and capacity"
description: Capacity uses workload-specific objectives, manifest and deployment safety budgets, effective mutation accounting, and benchmark validation.
tags: [architecture, adr, performance, capacity, throughput, latency]
status: accepted
decision_id: ADR-0020
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Live freshness targets are initially p95 <= 30 seconds and p99 <= 2 minutes for accepted events under normal load.
  - A one-hour outage must be recoverable within two hours after dependencies recover, subject to benchmark validation.
  - Hard safety limits protect queues, memory, documents, nested items, fan-out, batches, source reads, and effective mutations; performance targets generate alerts rather than silent drops.
  - Sustainable capacity is planned at approximately two times peak effective mutation rate until representative benchmarks establish a better margin.
  - Unsafe or unmeasured manifests are rejected or blocked before production use; oversized records remain diagnosable even while their terminal disposition is deferred.
---

# ADR-0020: Performance and capacity

## Context

The draft contains example batch sizes, timeouts, and worker counts, but event
cost varies with relationship fan-out, document size, cache misses, and the
number of dual-write targets. Existing decisions require bounded queues,
pending work, nested cardinality, and source-read protection, but do not define
the measurable workload envelope or recovery headroom.

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
memory per partition, controlled source-read rate, and effective mutations per
event. Exceeding a hard limit pauses, rejects, defers, or otherwise isolates the
affected work under the existing delivery and DLQ rules. Throughput, lag,
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

Oversized Debezium records must emit an explicit metric, structured diagnostic,
and identifiable audit record even though their final terminal disposition is a
follow-up decision.

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

## Review triggers

Revisit when benchmark results contradict the initial targets, workload shape or
fan-out changes materially, recovery objectives change, or limits threaten
capacity or search freshness.

## Related concepts

- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
