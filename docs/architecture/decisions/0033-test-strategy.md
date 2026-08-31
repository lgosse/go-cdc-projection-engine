---
type: Architecture Decision Record
title: "ADR-0033: Test strategy"
description: Layered per-change, integration, and release testing proves accepted contracts and correctness while scheduled resilience testing is deferred.
tags: [architecture, adr, validation, testing, contracts]
status: accepted
decision_id: ADR-0033
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Fast per-change tests, focused real-dependency integration tests, and release-level end-to-end tests are required initially.
  - Nightly or scheduled resilience, soak, and broad fault-injection testing is a later validation topic and is not a first-release gate.
  - Tests map to accepted correctness invariants and distinguish match, mismatch, and unknown outcomes.
  - Production fixtures are synthetic or sanitized and preserve no unauthorized raw payloads.
  - Required and production dependency capabilities are tested against the compatibility matrix.
---

# ADR-0033: Test strategy

## Context

The engine's accepted correctness invariants cover duplicate and reordered
histories, deletion fences, offset custody, canonical convergence, migration
cutover, and explicit unknown comparison outcomes. Store and protocol behavior
also depends on real Kafka, MongoDB, Redis, Elasticsearch, Kubernetes, and
embedded runtime capabilities. A single test style cannot provide fast feedback
and credible release evidence at the same time.

## Decision

Use three required initial layers.

Fast per-change tests cover formatting and static analysis, unit behavior,
manifest schema and semantic validation, CDC envelope contracts,
canonicalization, identity, ordering, transformations, error classification,
deterministic property histories, and `match`/`mismatch`/`unknown` comparison
outcomes. They should not require the complete dependency stack.

Focused integration tests run when a change affects stores, protocols,
transformation runtime, offsets, or writes. They use real Kafka, MongoDB, Redis,
and Elasticsearch instances and exercise bulk responses, mappings, aliases,
scripts, expiry, change streams, commits, DLQ custody, checkpoints, leases, and
fencing against required and production versions in the compatibility matrix.

Release-level end-to-end tests exercise streaming, boundary-overlap bootstrap
and live dual-write, relationship disorder, deletes, replay, reconciliation,
crash recovery, schema migration, verification, alias cutover, rollback, and
retirement safeguards. Fixtures are synthetic or sanitized and do not contain
unrestricted production payloads.

Use a small deterministic reference model for event histories and canonical
projection state. An `unknown` comparison caused by unavailable source evidence
does not trigger repair or failure solely for that reason.

Fast-test failures block merging. Required or production dependency capability
failures block release. End-to-end migration, rollback, offset-custody, and
deletion-safety failures block production promotion.

Broad scheduled resilience work—outage and latency injection, rebalances, pod
termination, Redis loss, interrupted bootstrap or migration, soak, catch-up,
disaster recovery, and privacy-erasure exercises—is explicitly deferred to a
later validation topic and is not a first-release scheduled gate.

## Alternatives considered

1. **Unit and mocked integration tests only.** Fast, but unable to prove real
   store and protocol behavior.
2. **Run the complete four-store suite for every change.** Strong evidence but
   too slow and expensive for routine development.
3. **Use only end-to-end tests.** Broad failures arrive late and are difficult
   to diagnose.
4. **Use production snapshots as fixtures.** Realistic but unsafe for privacy
   and difficult to reproduce.

## Consequences

- Every accepted invariant maps to automated evidence.
- Isolated real-dependency environments are needed for integration and release
  testing.
- A reference model, fixture generator, and compatibility matrix are release
  responsibilities.
- First-release validation does not claim full scheduled resilience coverage.
- A later decision must define resilience cadence, environment, and gates.

## Validation

- Fast tests run on every change and cover affected contracts and invariants.
- Real supported dependencies exercise store and protocol behavior.
- The reference model agrees with implementation results for generated histories.
- Release tests demonstrate bootstrap, migration, rollback, replay, repair, and
  crash recovery.
- Fixtures contain no unauthorized production data.
- Deferred resilience scope is tracked explicitly rather than presented as a
  guarantee.

## Review triggers

Revisit when incidents expose missing resilience evidence, fixture distributions
prove unrepresentative, integration cost blocks delivery, or scheduled
resilience becomes a production-readiness requirement.

## Related concepts

- [Test strategy](../08-validation/test-strategy.md)
- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Performance and failure testing](../08-validation/performance-and-failure-testing.md)
- [Release acceptance](../08-validation/release-acceptance.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
