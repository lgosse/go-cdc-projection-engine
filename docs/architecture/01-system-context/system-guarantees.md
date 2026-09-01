---
type: Architecture Review Topic
title: System guarantees
description: Defines the externally meaningful correctness and availability promises.
tags: [context, guarantees, semantics]
sources:
  - resource: ../../design/system.md
    title: System design draft
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

# System guarantees

## Decision

### Delivery and correctness

The engine provides at-least-once processing; exactly-once processing is not
promised. Every committed Kafka offset has a definitive terminal outcome for
each required target. Duplicate, delayed, and out-of-order events converge
deterministically when they have valid identity and source-revision metadata,
remain within the supported retention horizon, and required source evidence is
available. Older revisions cannot overwrite newer revisions or resurrect fenced
deletes. Malformed or unsupported event-local records receive durable DLQ
custody rather than silently disappearing.

### Freshness and recovery

Under normal load and available dependencies, accepted events become searchable
within p95 30 seconds and p99 2 minutes. A one-hour outage backlog should be
recoverable within two hours after dependencies recover, subject to benchmark
evidence. These are objectives, not promises during dependency outages,
retention loss, malformed events, blocked projections, or deliberate maintenance.

### Migration availability

“Zero downtime” means the search alias always points to one verified physical
target during a compatible migration, live dual-write is active before cutover,
alias movement is atomic, and the old target remains protected during the
rollback window. No planned read outage is introduced by migration. Availability
during Elasticsearch or platform-wide outages is outside this guarantee.

### Rebuildability and source access

Redis and Elasticsearch are rebuildable derived state. MongoDB is canonical for
bootstrap, reconciliation, and repair. Stream processing does not perform
unrestricted foreign-database point reads; controlled source fallback is allowed
only for explicitly allow-listed relation policies.

### Mode-specific promises

- `stream` provides live Kafka processing and the freshness objectives.
- `bootstrap` converges to MongoDB-derived state using the accepted source
  boundary and overlap replay without owning live Kafka offsets.
- `migration` exposes only verified targets and preserves rollback evidence.
- `audit` compares state; `repair` is explicit, rate-limited, and fenced.
- `dlq replay` reprocesses selected durable DLQ records idempotently without
  pretending they were ordinary Kafka progress.

### Exceptions

Guarantees become conditional or are suspended when dependencies are unavailable,
Kafka history or deletion evidence has expired, source boundaries or authority
are unavailable, envelopes are malformed or unsupported, manifests or
capabilities are invalid, hard safety limits are exceeded, or an operator
deliberately pauses work. The system reports the affected scope and reason rather
than claiming success.

## Pros

- Creates verifiable service-level semantics without implying impossible
  exactly-once behavior.
- Makes convergence assumptions, freshness objectives, and migration availability
  explicit.
- Distinguishes migration continuity from external platform availability.
- Gives operators and clients clear exception boundaries.

## Cons and risks

- Conditional guarantees depend on retention, source evidence, and dependency
  availability being observable.
- At-least-once delivery moves idempotency complexity into sink operations.
- A planned read-outage guarantee still requires external Elasticsearch SLOs.
- Projection-specific consumers may need stricter contracts than engine defaults.

## Alternatives considered

1. Promise exactly-once processing. Separate Kafka, MongoDB, Redis, and
   Elasticsearch transactions cannot support this claim.
2. Promise convergence after arbitrary reordering. This is indefensible when
   revisions, retention, or deletion evidence are missing.
3. Define zero downtime as universal availability. This confuses migration
   behavior with external platform outages.
4. Promise freshness regardless of dependencies. This encourages unsafe retries
   or silent data loss.

## Consequences

- Documentation distinguishes guarantees, objectives, preconditions, and
  non-guarantees.
- Clients and operators need scoped status when guarantees become conditional.
- Retention and source-boundary evidence become part of convergence validation.
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

## Review trigger

Revisit if a guarantee cannot be tested, retention or source boundaries change,
freshness or recovery objectives are missed, migration causes planned read
outages, or consumers require stronger semantics.

## Related concepts

- [System guarantees](system-guarantees.md)
- [Objectives and boundaries](objectives-and-boundaries.md)
- [Data-store responsibilities](data-store-responsibilities.md)
- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Live ingestion source](live-ingestion-source.md)
- [Offsets and delivery](../03-runtime/offsets-and-delivery.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
