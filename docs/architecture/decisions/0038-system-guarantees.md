---
type: Architecture Decision Record
title: "ADR-0038: System guarantees"
description: The engine promises bounded at-least-once delivery, conditional convergence, measurable freshness, verified migration continuity, and explicit scoped exceptions.
tags: [architecture, adr, context, guarantees, semantics]
status: accepted
decision_id: ADR-0038
accepted_on: 2026-08-31
owner: TBD
conditions:
  - The engine promises at-least-once processing, not exactly-once processing.
  - Deterministic convergence applies only to supported histories with valid identity/revision metadata, retained evidence, and available authorities.
  - Initial freshness objectives are p95 <= 30 seconds and p99 <= 2 minutes under normal load; a one-hour outage should catch up within two hours after dependencies recover, subject to benchmark evidence.
  - Zero downtime means no planned read outage during a compatible migration; it does not promise availability during platform-wide or Elasticsearch outages.
  - Exceptions are explicit and scoped for dependency outages, retention loss, unavailable source evidence, malformed events, invalid manifests, and hard safety limits.
---

# ADR-0038: System guarantees

## Context

The source draft uses aspirational terms such as “zero downtime” and
“order-agnostic.” Accepted decisions now define at-least-once offsets,
source-scoped fencing, deletion custody, MongoDB canonical state, versioned
migrations, measurable freshness and recovery targets, and explicit correctness
invariants. These need to become one bounded external contract.

## Decision

Promise at-least-once processing, not exactly-once processing. Every committed
Kafka offset has a definitive terminal outcome for each required target. Duplicate,
delayed, and out-of-order events converge deterministically when identity and
source-revision metadata are valid, retained evidence is available, and required
authorities can be read. Older revisions cannot overwrite newer revisions or
resurrect fenced deletes. Malformed or unsupported event-local records receive
durable DLQ custody.

Under normal load and available dependencies, accepted events become searchable
within p95 30 seconds and p99 2 minutes. A one-hour outage backlog should be
recoverable within two hours after dependencies recover, subject to benchmark
evidence. These are objectives, not promises during dependency outages,
retention loss, malformed events, blocked projections, or deliberate maintenance.

“Zero downtime” means that a compatible migration keeps the search alias on one
verified physical target, activates live dual-write before cutover, moves the
alias atomically, and protects the old target during the rollback window. It does
not promise availability during Elasticsearch or platform-wide outages.

Redis and Elasticsearch are rebuildable derived state; MongoDB is canonical for
bootstrap, reconciliation, and repair. Stream processing does not perform
unrestricted foreign-database point reads. Controlled source fallback is allowed
only for explicitly allow-listed relation policies.

Mode promises are explicit: stream provides live Kafka processing; bootstrap
converges to MongoDB-derived state using the accepted boundary and overlap replay;
migration exposes only verified targets; audit compares; repair is explicit,
rate-limited, and fenced; and DLQ replay is idempotent without advancing Kafka
progress.

Guarantees become conditional or are suspended when dependencies are unavailable,
history or deletion evidence has expired, source boundaries are unavailable,
events are malformed or unsupported, manifests/capabilities are invalid, hard
safety limits are exceeded, or an operator deliberately pauses work. The system
reports scope and reason rather than claiming success.

## Alternatives considered

1. **Promise exactly-once processing.** Separate Kafka, MongoDB, Redis, and
   Elasticsearch transactions cannot support this claim.
2. **Promise convergence after arbitrary reordering.** This is indefensible
   when revisions, retention, or deletion evidence are missing.
3. **Define zero downtime as universal availability.** This confuses migration
   behavior with external platform outages.
4. **Promise freshness regardless of dependencies.** This encourages unsafe
   retries or silent data loss.

## Consequences

- Documentation distinguishes guarantees, objectives, preconditions, and
  non-guarantees.
- Clients and operators need scoped status when guarantees become conditional.
- Retention and source-boundary evidence are part of convergence validation.
- Projection manifests may define stricter objectives than engine defaults.

## Validation

- Contract tests demonstrate at-least-once behavior, terminal offset custody,
  duplicate handling, fencing, and delete safety.
- Generated histories converge under documented valid preconditions.
- Freshness and recovery objectives hold under representative benchmarks.
- Migration tests show uninterrupted alias availability, verified cutover, and
  rollback.
- Dependency, retention, malformed-event, and hard-limit exceptions produce
  explicit scoped status.
- Documentation distinguishes guarantees, objectives, preconditions, and
  non-guarantees.

## Review triggers

Revisit if a guarantee cannot be tested, retention or source boundaries change,
freshness or recovery objectives are missed, migration causes planned read
outages, or consumers require stronger semantics.

## Related concepts

- [System guarantees](../01-system-context/system-guarantees.md)
- [Objectives and boundaries](../01-system-context/objectives-and-boundaries.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Live ingestion source](../01-system-context/live-ingestion-source.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
