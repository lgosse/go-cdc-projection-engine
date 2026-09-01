---
type: Architecture Review Topic
title: Startup and readiness
description: Defines safe startup validation and degraded-operation behavior.
tags: [runtime, startup, readiness, schema]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0042
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Each mode has an explicit preflight checklist and failure-scope matrix before implementation.
  - No source progress occurs until the relevant startup checks pass.
  - Recoverable post-start dependency failures normally pause or degrade affected work; readiness becomes negative only when safe participation cannot be maintained.
  - Termination uses a bounded drain and never commits after partition revocation.
---

# Startup and readiness

## Decision

Run a deterministic preflight before source progress. For the selected mode,
validate immutable configuration, manifest syntax and semantics, relationship
constraints, field classifications, transformation compilation, required
dependency capabilities, engine-owned MongoDB metadata and migration state, and
the relevant Elasticsearch mappings, aliases, scripts, and active write targets.
Do not consume Kafka records or begin a bounded mutating operation until the
applicable checks pass.

Treat invalid global configuration, missing required capabilities, and unsafe
control-plane state as workload failures: the workload remains unready and must
not make source progress. An isolated projection or target may be blocked while
other projections continue only when partition coupling and the blocked
projection's effect on shared offset progress remain explicit and safe.

Expose local-process liveness separately from workload readiness. Liveness is
local-only and does not depend on Kafka, MongoDB, Redis, or Elasticsearch.
Readiness indicates whether the pod can safely participate in assigned work.
After startup, temporary dependency failures normally produce scoped `degraded`
or `paused` status rather than immediately making every pod unready; readiness
becomes negative for invalid startup state, unrecoverable local failure, or an
explicitly unsafe participation state.

During Kubernetes termination, stop intake, drain bounded in-flight work within
a configured deadline, commit only safely completed contiguous offsets, and then
relinquish participation. A revoked partition cannot be committed by its
previous owner.

This decision does not define the exact mode-by-mode checklist, the final
projection/partition/workload failure matrix, or the numeric drain and
revalidation intervals; those are explicit follow-up contracts.

## Rationale and benefits

- Prevents consumption under a known-invalid configuration.
- Separates a live process from one able to make projection progress.
- Projection isolation can preserve healthy workloads.

## Alternatives considered

1. Make readiness fail whenever any dependency or projection is degraded. This
   is simple, but temporary Elasticsearch or Redis failures can cause pod
   removal, Kafka rebalances, and duplicate processing without improving
   correctness.
2. Consume first and pause after validation. This risks delivery progress before
   the worker has proven that it can safely process the event.
3. Use only one global health result. This hides which projection or partition
   is blocked and makes safe partial operation difficult to observe.

## Consequences and risks

- Live mapping comparison is more complex than textual equality.
- Pausing shared topics can accidentally block unrelated projections.
- Continuous polling can hide a condition that needs operator action.

## Follow-up questions

- Which checks are mandatory for each mode (`stream`, `bootstrap`, `repair`,
  `replay`, and `migration`)?
- If a running worker loses engine-owned MongoDB connectivity, should it remain
  ready-but-paused for a bounded grace period, or become unready immediately
  because safe participation cannot be confirmed?
- What exact rule determines whether a failure blocks one projection, a Kafka
  partition, a workload group, or the whole deployment?
- What revalidation cadence and bounded termination deadline should be used?

## Validation

- Failed preflight prevents source consumption or unsafe maintenance progress.
- Projection-level blocking never hides shared partition coupling.
- Dependency outages pause or degrade safely without restart storms.
- Termination and rebalances preserve contiguous offset correctness.
- Probe responses and diagnostics agree on failure scope and observation time.
