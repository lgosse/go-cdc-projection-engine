---
type: Architecture Decision Record
title: "ADR-0042: Startup and readiness"
description: Defines deterministic preflight validation and safe Kubernetes probe behavior.
tags: [architecture, adr, runtime, startup, readiness]
status: accepted
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0042: Startup and readiness

## Status

Accepted on 2026-08-31. Owner: TBD.

## Context

The source boot sequence validates manifests, transformations, dependencies,
and Elasticsearch mappings before Kafka subscription, while later decisions
require scoped projection failures, partition-contiguous progress, and stable
Kubernetes health behavior. A probe that treats every dependency outage as a
pod failure would create restart and rebalance storms; consuming before
validation could advance delivery state under an invalid target or manifest.

## Decision

Run a deterministic, mode-specific preflight before source progress. Validate
immutable configuration, manifest syntax and semantics, relationship
constraints, field classifications, transformation compilation, required
dependency capabilities, engine-owned MongoDB metadata and migration state,
and relevant Elasticsearch mappings, aliases, scripts, and active write
targets. Do not consume Kafka records or begin a bounded mutating operation
until applicable checks pass.

Invalid global configuration, missing required capabilities, and unsafe
control-plane state keep the workload unready and prevent source progress. An
isolated projection or target may be blocked while other projections continue
only when partition coupling and the blocked projection's effect on shared
offset progress remain explicit and safe.

Liveness is local-process-only. Readiness indicates safe participation in
assigned work. Temporary post-start dependency failures normally produce
scoped `degraded` or `paused` status rather than immediately making every pod
unready. Readiness becomes negative for invalid startup state, unrecoverable
local failure, or an explicitly unsafe participation state.

On Kubernetes termination, the worker stops intake, drains bounded in-flight
work within a configured deadline, commits only safely completed contiguous
offsets, and relinquishes participation. A revoked partition cannot be
committed by its previous owner.

The exact mode checklists, failure-scope matrix, revalidation cadence, and
numeric drain deadline remain follow-up contracts. This ADR does not define
implementation or deployment manifests.

## Alternatives considered

1. **Fail readiness on every degradation.** Simpler, but causes unnecessary
   pod removal, Kafka rebalances, and duplicate work during transient sink or
   cache outages.
2. **Consume before validation.** Risks delivery progress before safe target
   and manifest validation.
3. **One global health result.** Hides projection and partition scope and makes
   safe partial operation difficult to diagnose.

## Consequences

- Startup failures fail closed and are visible before source progress.
- Healthy projections can continue only when shared partition consequences are
  represented explicitly.
- Health consumers need separate liveness, readiness, ingestion, recovery,
  and per-projection status.
- Preflight adds startup latency and requires maintained mode-specific checks.
- Graceful termination and rebalance draining are correctness contracts.
- Operators must distinguish ready-but-paused from healthy progress using
  diagnostics and alerts.

## Validation

- Failed preflight prevents source consumption or unsafe maintenance progress.
- Projection-level blocking never hides shared partition coupling.
- Dependency outages pause or degrade safely without restart storms.
- Termination and rebalances preserve contiguous offset correctness.
- Probe responses and diagnostics agree on failure scope and observation time.

## Review triggers

Revisit if readiness flaps during dependency incidents, failure scope cannot be
represented safely, startup checks become too slow for operations, or graceful
termination cannot drain within the supported Kafka rebalance window.

## Related concepts

- [Startup and readiness](../03-runtime/startup-and-readiness.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Runtime topology](../03-runtime/runtime-topology.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
