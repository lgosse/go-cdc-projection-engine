---
type: Architecture Decision Record
title: "ADR-0056: Nested relation capacity boundaries"
description: Separates projection-author expectations from engine-enforced nested-document safety limits.
tags: [architecture, adr, elasticsearch, nested, capacity, contracts]
status: accepted
decision_id: ADR-0056
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Projection authors choose nested versus separate child projection and declare expected cardinality for planning and warning.
  - Crossing an expected bound alone raises an alert but does not reject valid source events while the projection remains within its measured safe envelope.
  - Engine-supported hard limits are established from benchmarks and target capabilities; manifests beyond them are blocked before use.
  - Bootstrap cannot cut over a target that exceeds a hard limit; live work at a hard limit is blocked at the smallest safe scope and durably preserved, not DLQed as poison data.
  - Numeric limits remain open until representative capacity evidence exists (Q-070).
---

# ADR-0056: Nested relation capacity boundaries

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

V1 uses bounded nested arrays for one-to-many child relations, while large
relations may need separate projections or indices. Projection authors know
the domain's expected volume and query needs; the engine knows the tested
capacity envelope of its supported runtime and target configuration. A single
declared expectation must not be confused with a technical safety limit.

There is no implementation or representative workload benchmark yet. Therefore
this decision sets the ownership and enforcement boundary, not a numeric child
count. Numeric capacity limits are established through the benchmark work in
Q-070.

## Decision

- The projection author chooses nested children or a separately designed child
  projection/index and declares the expected maximum cardinality and relevant
  document-size estimate. This declaration supports design review, capacity
  planning, and early warning; it is not by itself a reason to discard valid
  source data.
- The engine publishes a supported hard envelope based on representative
  benchmarks and target capabilities. Manifest preflight rejects a declared
  configuration that exceeds that envelope before Kafka consumption or source
  scanning begins. Preflight validates the declaration; it does not claim to
  know the actual source cardinality without scanning the source.
- During bootstrap, measure actual relation cardinality and projected document
  size. Exceeding the author's expectation raises an alert. If the target
  remains inside the supported hard envelope, bootstrap may continue. If the
  hard envelope is exceeded or the target cannot be shown safe, keep the new
  target from cutover and report the blocking measurement.
- During live processing, crossing the declared expectation raises an alert
  and processing continues while the candidate mutation remains inside the
  supported hard envelope. At the hard boundary, block the smallest safe
  projection/document scope and durably preserve valid source work for recovery;
  do not classify that work as poison data or silently drop it. Exact queue,
  pause, and offset behavior follows the runtime decisions tracked by Q-019,
  Q-021, and Q-032.
- Do not switch a live relation from nested storage to a separate index
  automatically. That changes target mappings, query composition, and migration
  behavior; it requires a developer-authored manifest change and the relevant
  migration procedure.
- Establish numeric warning and hard limits from benchmarks before production
  use (Q-070). Until those limits are evidenced, do not claim an unmeasured
  manifest is production-safe.

## Example

Suppose a projection author estimates at most 1,200 children, while a later
representative benchmark establishes a hard safe cap of 5,000 for that relation
and deployment. At child 1,201, the engine alerts and keeps processing. At a
prospective child count of 5,001, the engine holds that affected work and does
not publish an unsafe target. These figures are illustrative only; Q-070 must
establish actual values.

## Alternatives considered

1. **Treat the author's expected count as a hard limit.** This is easy to
   validate, but a reasonable estimate error could reject valid source data
   even when the target remains safe.
2. **Use alerts only with no engine hard boundary.** This leaves Elasticsearch
   or process resource limits to fail unpredictably, potentially creating
   retries, backpressure, or partial progress during production traffic.
3. **Let the engine switch storage shape automatically.** This appears
   transparent but changes index mappings and query behavior without an
   approved migration or consumer contract. This is rejected.
4. **Separate author expectation from a benchmarked engine safety cap.** This
   gives authors ownership of the model, permits safe growth beyond estimates,
   and still blocks unsafe work. It is accepted.

## Consequences

- Authors own cardinality expectations and the nested-versus-separate modeling
  choice; the engine owns validation and safe enforcement.
- Warnings can arrive before any pause, giving the owner time to review data
  growth and plan a migration.
- A hard-limit breach can block affected work and reduce freshness until the
  projection is redesigned or capacity is safely increased. It must remain
  visible and recoverable.
- Bootstrap can validate observed cardinality before alias cutover. Live
  enforcement needs a bounded way to retain work and isolate the affected
  scope, specified by the runtime follow-ups.
- The same qualitative trigger resolves Q-071: move a relation to a separate
  projection when it exceeds the tested hard envelope for cardinality,
  document size, or required update/search performance. Numeric limits remain
  under Q-070.

## Validation

- A manifest's declared expectation is compared with the engine's published
  hard envelope before source progress; no source scan is implied by preflight.
- Crossing only the author expectation produces an alert and the valid event
  continues if the candidate mutation remains within the hard envelope.
- Bootstrap measures actual counts and size and does not cut over a target that
  exceeds the hard envelope.
- A live event that would exceed the hard envelope is durably preserved and
  blocked at the smallest safe scope without being marked as poison data.
- The engine never silently truncates the child array or changes it to another
  index layout at runtime.

## Review triggers

Revisit when representative benchmarks establish new safe limits, workloads
show warning thresholds provide too little response time, or operational
evidence shows the chosen block scope causes unacceptable partition impact.

## Related decisions and concepts

- [ADR-0020: Performance and capacity](0020-performance-and-capacity.md)
- [ADR-0042: Startup and readiness](0042-startup-and-readiness.md)
- [ADR-0048: Event size and buffer diagnostics](0048-event-size-and-buffer-diagnostics.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Follow-up register](../follow-ups.md) (Q-012, Q-019, Q-021, Q-032, Q-070, Q-071)
