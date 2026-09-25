---
type: Architecture Review Topic
title: Performance and failure testing
description: Defines benchmark, soak, rebalance, and fault-injection evidence.
tags: [validation, performance, chaos, resilience]
status: accepted
decision_id: ADR-0034
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Benchmarks use representative projection profiles rather than one global throughput number.
  - Targeted failure tests validate accepted correctness and recovery guarantees before production.
  - Destructive migration and recovery tests run only in isolated Kubernetes environments with disposable targets and data.
  - Initial release gates cover freshness, recovery, hard safety limits, capabilities, and correctness; recurring chaos and soak campaigns remain deferred.
  - Synthetic or sanitized fixtures are generated from documented workload profiles.
---

# Performance and failure testing

## Decision

Benchmark representative workload profiles rather than one global throughput
number. Profiles include root-only projections, reference and multi-hop lookups,
low- and high-cardinality nested arrays, high fan-out, hot-key skew, cache hits
and misses, expensive transformations, duplicate and out-of-order delivery,
blue-green dual-write, and large or oversized records.

Measure effective mutations per second, event-to-searchable freshness
percentiles, bootstrap and catch-up duration, queue depth and age,
Elasticsearch bulk latency and throttling, Redis and MongoDB read latency, CPU,
memory, retries, backpressure, and document/nested-item/fan-out/batch/source-read
limits. For transformations, include mapping compile complexity, event and
canonical input/output bytes, array items traversed, evaluation CPU/latency, and
peak worker memory under concurrency
([ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md)).

Use four benchmark phases:

1. **Nominal:** expected production workload.
2. **Peak:** expected maximum sustained workload.
3. **Stress:** approach hard safety limits without corrupting data.
4. **Recovery:** process a one-hour outage backlog after dependencies recover.

Before production, run targeted failure tests for Kafka redelivery and rebalance,
pod termination during processing, partial Elasticsearch bulk failure, dependency
latency and outage, Redis loss and rebuild, interrupted bootstrap, migration
restart and rollback, and DLQ and repair behavior. Assert the intended failure
scope for each scenario: event-local, partition-scoped, projection-scoped, or
workload-wide. Shared-metadata global-stop criteria remain deferred under
ADR-0032.

Run destructive migration and recovery tests in isolated Kubernetes environments
with disposable indices, databases, aliases, and data. Use synthetic or sanitized
fixtures generated from documented workload profiles.

Initially block production promotion on absolute invariant violations, unsafe
handling of hard limits, missed accepted freshness or recovery objectives under
representative profiles, unsafe migration/rollback/recovery, or required
dependency capability failures. Scheduled soak, nightly chaos, and broad
recurring resilience campaigns remain deferred until a later decision defines
their cadence and ownership.

## Pros

- Measures fan-out, nested cardinality, hot keys, cache behavior, and dual-write
  cost instead of relying on average throughput.
- Validates failure scope and recovery against accepted correctness guarantees.
- Makes destructive testing safe and reproducible.
- Provides release evidence without requiring a permanent chaos-testing program.

## Cons and risks

- Representative profiles and synthetic data generators require maintenance.
- Multi-store benchmark environments are expensive and can be noisy.
- Strict release gates may require high-fan-out projections to be split or
  capacity-isolated.
- Deferred recurring resilience testing leaves an operational evidence gap.
- Performance can regress materially while remaining within an objective unless
  a regression margin is later defined.

## Alternatives considered

1. Benchmark only average throughput. This hides p99 freshness, hot-key skew,
   fan-out, memory pressure, and dependency saturation.
2. Replay production traffic. This is realistic but creates privacy,
   reproducibility, and operational-safety concerns.
3. Test only happy paths. This cannot validate retries, rebalances, partial
   writes, or recovery.
4. Require full recurring chaos testing before first release. This increases
   confidence but conflicts with the accepted deferral of scheduled resilience
   campaigns.

## Consequences

- Benchmark results become maintained release evidence.
- Isolated Kubernetes environments and disposable targets are required for
  destructive tests.
- Release gates must record workload profile, dependency versions, limits, and
  measured percentiles.
- High-fan-out or high-cardinality projections may require separate capacity
  boundaries.
- A later decision must define recurring resilience cadence, ownership, and
  regression margins.

## Validation

- Representative profiles meet freshness, throughput, safety-limit, and recovery
  objectives.
- High fan-out, hot keys, nested data, cache misses, and dual-write are measured.
- Targeted failure tests demonstrate intended failure scope and safe recovery.
- No destructive test touches production data, aliases, or targets.
- Results are reproducible and retained with release evidence.

## Review trigger

Revisit when workloads, fan-out, dependency behavior, or recovery objectives
change materially, when release evidence is not reproducible, or when recurring
resilience testing becomes a production-readiness requirement.

## Related concepts

- [Performance and failure testing](performance-and-failure-testing.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Correctness invariants](correctness-invariants.md)
- [Test strategy](test-strategy.md)
- [Release acceptance](release-acceptance.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
