---
type: Architecture Review Topic
title: Test strategy
description: Defines layered verification for manifests, event histories, stores, and operational workflows.
tags: [validation, testing, contracts]
status: accepted
decision_id: ADR-0033
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Fast per-change tests, focused real-dependency integration tests, and release-level end-to-end tests are required initially.
  - Nightly or scheduled resilience, soak, and broad fault-injection testing is a later validation topic and is not a first-release gate.
  - Tests map to the accepted correctness invariants and distinguish match, mismatch, and unknown outcomes.
  - Production fixtures are synthetic or sanitized and preserve no unauthorized raw payloads.
  - Required and production dependency capabilities are tested against the compatibility matrix.
---

# Test strategy

## Decision

Use layered verification.

### Fast per-change tests

Run for every change: formatting and static analysis; unit tests; manifest schema
and semantic validation for every manifest; CDC envelope contract tests;
canonicalization, identity, ordering, transformation, and error-classification
tests; deterministic property tests over duplicate, reordered, delayed, deleted,
and replayed histories; and tests for `match`, `mismatch`, and `unknown`
reconciliation outcomes.

These tests should not require the complete dependency stack.

### Focused integration tests

Run when a change affects a store, protocol, transformation runtime, offsets, or
writes. Use real Kafka, MongoDB, Redis, and Elasticsearch instances to exercise
bulk responses, mappings, aliases, scripts, expiry, change streams, commits,
DLQ custody, checkpoints, leases, and fencing. Test the required and production
versions from the compatibility matrix. Mocks may support unit tests but are not
sufficient evidence for store or protocol behavior.

### Release-level end-to-end tests

Before production promotion, exercise complete stream processing, bootstrap with
boundary overlap and live dual-write, child-before-parent and missing-parent
behavior, deletes, replay, reconciliation repair, crash recovery, schema
migration, verification, alias cutover, rollback, and retirement safeguards.

Use synthetic or sanitized fixtures generated to resemble production
distributions. Do not use unrestricted production payloads as test data.

### Deferred resilience testing

Broad property histories, dependency latency and outage injection, partial bulk
failures, rebalances, pod termination, Redis loss and rebuild, interrupted
bootstrap and migration, soak, catch-up, disaster recovery, and privacy-erasure
exercises are deferred to a later validation topic. They are not required as
first-release scheduled gates, but accepted invariants still require focused
failure tests in the per-change, integration, or release layers where relevant.

### Test oracle

Use a small deterministic reference model for event histories and canonical
projection state. Compare implementation results with that model rather than
relying only on document counts or final Elasticsearch contents. For
cross-service comparisons, preserve `match`, `mismatch`, and `unknown`; an
`unknown` result must not trigger repair or failure solely because source
evidence is unavailable.

### Release blocking

Fast per-change failures block merging. Required and production dependency
capability failures block release. End-to-end migration, rollback, offset
custody, and deletion-safety failures block production promotion. Scheduled
resilience regressions become release blockers only after the deferred resilience
topic defines them and when they affect an accepted invariant or objective.

## Pros

- Matches each test layer to the boundary it can actually prove.
- Keeps routine development practical without weakening release evidence.
- Real-store tests catch script, mapping, offset, alias, expiry, and protocol
  behavior that mocks miss.
- Generated histories exercise correctness claims systematically.

## Cons and risks

- A four-store integration suite is slow and operationally heavy.
- Property tests need a trustworthy reference model.
- Synthetic distributions may miss unknown production shapes.
- Deferred resilience testing leaves a first-release operational evidence gap.
- Version-matrix testing can dominate release time.

## Alternatives considered

1. Unit and mocked integration tests only. Fast, but unable to prove real store
   and protocol behavior.
2. Run the complete four-store suite for every change. Strong but too slow and
   expensive for routine development.
3. Use only end-to-end tests. Broad failures arrive late and are hard to
   diagnose.
4. Use production snapshots as fixtures. Realistic but unsafe for privacy and
   difficult to reproduce.

## Consequences

- Every accepted invariant must map to automated evidence.
- Ephemeral or otherwise isolated real-dependency environments are needed for
  focused integration and release tests.
- A reference model, fixture generator, and compatibility matrix become release
  responsibilities.
- First-release validation does not claim full scheduled resilience coverage.
- A later resilience decision must define its cadence, environment, and gates.

## Validation

- Fast tests run on every change and cover all affected contracts and invariants.
- Real supported dependencies exercise store and protocol behavior.
- The reference model agrees with implementation results for generated histories.
- Release tests demonstrate bootstrap, migration, rollback, replay, repair, and
  crash recovery.
- Test fixtures contain no unauthorized production data.
- Deferred resilience scope is tracked as an explicit follow-up rather than an
  implicit guarantee.

## Review trigger

Revisit when first-release incidents expose missing resilience evidence, fixture
distributions prove unrepresentative, integration cost blocks delivery, or the
deferred resilience topic becomes a production-readiness requirement.

## Related concepts

- [Correctness invariants](correctness-invariants.md)
- [Performance and failure testing](performance-and-failure-testing.md)
- [Release acceptance](release-acceptance.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
