---
type: Architecture Decision Record
title: "ADR-0027: Health and diagnostics"
description: Liveness is local-only, readiness is stable and scoped, and authenticated diagnostics expose authoritative projection state with timestamps.
tags: [architecture, adr, observability, health, readiness, diagnostics]
status: accepted
decision_id: ADR-0027
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Liveness is local-process-only; dependency outages do not restart healthy workers.
  - Readiness covers startup validation, required capabilities, local control-loop health, and safe participation in assigned work.
  - Search-read health, ingestion readiness, recovery/backlog state, and per-projection status are exposed separately.
  - Diagnostics are authenticated, bounded, timestamped, and omit raw payloads, secrets, Restricted fields, and unprotected identifiers.
  - Engine-owned MongoDB is authoritative for control-plane state; diagnostics identify source and observation time when backend views disagree.
---

# ADR-0027: Health and diagnostics

## Context

The engine must keep search availability independent from projector worker
health, while Kubernetes probes must not create restart or rebalance storms
during ordinary dependency outages. Operators also need projection-level state
that is safer and more reliable than inferring it from logs.

## Decision

Expose a local-process-only liveness endpoint that answers whether the process
can run its event loop. It must not require Kafka, MongoDB, Redis, or
Elasticsearch, so ordinary dependency outages do not trigger restart storms.

Expose a readiness endpoint that verifies configuration and manifest validation,
required capability checks, local worker/control-loop health, required control-
plane visibility, and absence of unrecoverable process invariants. Temporary
dependency outages normally produce a degraded or paused state rather than make
every worker unready and cause rebalance churn. Readiness becomes negative for
startup failures, invalid configuration, unrecoverable local state, or an
explicitly unsafe condition.

Expose separate search-read, ingestion, recovery/backlog, and per-projection
status. A projection may be blocked while another remains healthy; shared Kafka
partition coupling must still be reported rather than hidden.

Provide an authenticated, bounded diagnostic view containing projection and
manifest identity, configuration hash, assigned partitions, paused/blocked
reason, source-watermark age, Kafka lag, active write targets, migration phase,
last successful flush, retry/terminal summaries, cache-generation state, repair
state, and observation timestamps. Use DLQ/repair references or controlled
pseudonymous identifiers; never expose raw payloads, credentials, Restricted
fields, cache keys, or unprotected entity identifiers.

Diagnostics identify the authority for each observation: Kafka for assignment and
committed offsets, engine-owned MongoDB for migration/DLQ/fences/checkpoints,
Elasticsearch and Redis for target/dependency observations, and the process for
local health. Conflicting or stale views are reported with their source and
timestamp rather than silently reconciled. Probes are cheap, bounded, and do not
run full reconciliation or deep scans.

## Alternatives considered

1. Make readiness fail whenever any dependency or projection is degraded. This
   is simple but causes readiness flapping and restart/rebalance storms.
2. Use one global health endpoint without projection detail. This hides the
   affected workload and makes operator diagnosis depend on logs.
3. Perform deep dependency checks on every probe request. This produces fresher
   results but adds load and can make the probe itself a failure source.

## Consequences

- Search can remain available while ingestion is paused and freshness degrades.
- Readiness and diagnostics become stable, versioned operational interfaces.
- Status responses need freshness timestamps and explicit unknown/conflict
  states.
- Authenticated diagnostics require access control, redaction, and auditability.
- Shared partition coupling remains visible even when projections are otherwise
  isolated.

## Validation

- Dependency outages do not cause unnecessary restarts or rebalance storms.
- Invalid configuration prevents source consumption.
- Search remains available while ingestion is paused.
- Per-projection blocking is visible without exposing sensitive data.
- Diagnostics identify authoritative sources and observation times.
- Stale or conflicting backend state is reported explicitly.
- Readiness, alerts, and diagnostic status agree on the affected scope.

## Review triggers

Revisit if probes flap, diagnostics leak sensitive data, status freshness is
insufficient for operations, or health scopes cannot represent shared partition
coupling and migration states.

## Related concepts

- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
