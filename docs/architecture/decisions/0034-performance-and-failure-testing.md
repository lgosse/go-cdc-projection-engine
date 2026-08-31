---
type: Architecture Decision Record
title: "ADR-0034: Performance and failure testing"
description: Representative workload benchmarks and targeted failure tests validate capacity and recovery before production; recurring resilience campaigns are deferred.
tags: [architecture, adr, validation, performance, capacity, resilience]
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

# ADR-0034: Performance and failure testing

## Context

The engine's performance envelope depends on projection shape, relationship
fan-out, nested cardinality, hot keys, cache behavior, transformations,
dual-write targets, and dependency saturation. Accepted objectives include live
freshness, one-hour outage recovery, hard safety limits, and capacity planning by
effective mutations. The accepted test strategy requires targeted lifecycle and
failure evidence but defers recurring scheduled resilience campaigns.

## Decision

Benchmark representative profiles including root-only projections, reference and
multi-hop lookups, low- and high-cardinality nested arrays, high fan-out, hot-key
skew, cache hits and misses, expensive transformations, duplicate and
out-of-order delivery, blue-green dual-write, and large or oversized records.

Measure effective mutations per second, event-to-searchable freshness
percentiles, bootstrap and catch-up duration, queue depth and age,
Elasticsearch bulk latency and throttling, Redis and MongoDB read latency, CPU,
memory, retries, backpressure, and document/nested-item/fan-out/batch/source-read
limits. Exercise nominal, peak, stress, and one-hour-outage recovery phases.

Before production, run targeted failures for Kafka redelivery and rebalance, pod
termination, partial Elasticsearch bulk failure, dependency latency and outage,
Redis loss and rebuild, interrupted bootstrap, migration restart and rollback,
and DLQ and repair behavior. Assert whether each failure is event-local,
partition-scoped, projection-scoped, or workload-wide. Shared-metadata
global-stop criteria remain deferred under ADR-0032.

Run destructive migration and recovery tests only in isolated Kubernetes
environments with disposable indices, databases, aliases, and data. Fixtures are
synthetic or sanitized and generated from documented workload profiles.

Initially block production promotion on absolute invariant violations, unsafe
hard-limit handling, missed accepted freshness or recovery objectives under
representative profiles, unsafe migration/rollback/recovery, or required
dependency capability failures. Recurring soak, nightly chaos, and broad
resilience campaigns are deferred until a later decision defines their cadence,
ownership, and regression margins.

## Alternatives considered

1. **Benchmark only average throughput.** This hides p99 freshness, hot-key
   skew, fan-out, memory pressure, and dependency saturation.
2. **Replay production traffic.** Realistic but creates privacy,
   reproducibility, and operational-safety concerns.
3. **Test only happy paths.** Cannot validate retries, rebalances, partial
   writes, or recovery.
4. **Require recurring chaos testing before first release.** Increases confidence
   but conflicts with the accepted deferral of scheduled resilience campaigns.

## Consequences

- Benchmark results become maintained release evidence.
- Isolated Kubernetes environments and disposable targets are required for
  destructive tests.
- Release gates record workload profile, dependency versions, limits, and
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

## Review triggers

Revisit when workloads, fan-out, dependency behavior, or recovery objectives
change materially, release evidence is not reproducible, or recurring resilience
testing becomes a production-readiness requirement.

## Related concepts

- [Performance and failure testing](../08-validation/performance-and-failure-testing.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Test strategy](../08-validation/test-strategy.md)
- [Release acceptance](../08-validation/release-acceptance.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
