---
type: Architecture Review Topic
title: Source draft conflicts
description: Lists incompatible assumptions in the source drafts that require explicit decisions.
tags: [architecture, conflicts, ingestion, correctness]
sources:
  - resource: ../design/system.md
    title: System design draft
  - resource: ../design/otel.md
    title: OpenTelemetry design draft
status: accepted
decision_id: ADR-0043
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Foreign-database reads remain disabled by default and require an explicit relation policy.
  - Permitted fallback is bounded by deduplication, timeout, rate, source-protection, and cache-repopulation controls.
  - Required reverse-index misses default to rebuild, repair, or bounded pending rather than arbitrary synchronous reads.
---

# Source draft conflicts

## Decision

The source drafts are reconciled by treating “zero cross-database point queries”
as a prohibition on an unrestricted runtime query model, not a ban on every
source read. Normal stream processing uses Redis for relationship lookups. A
cache miss may synchronously read the authoritative MongoDB source only when the
relation's manifest policy explicitly permits it.

Permitted fallback requires per-key deduplication, timeout, rate limiting,
source-protection controls, and cache repopulation. Required reverse-index misses
default to asynchronous rebuild, repair, or bounded pending; synchronous lookup
is exceptional. If the source is unavailable, the engine does not invent a
value or publish an incomplete projection. It defers, pauses, repairs, or uses
the applicable durable custody path.

The other source conflicts are resolved by the linked accepted decisions for
live ingestion, durable state, the Debezium envelope, identity and ordering,
transformations, and offsets. Their detailed limits remain in those concepts'
follow-up questions.

## Rationale and benefits

- Matches the stated Kafka-first system boundary.
- Gives each datastore a clear role and failure contract.
- Avoids building two competing checkpoint and ingestion models.

## Alternatives considered

1. **Strict zero point reads:** defer every miss until cache rebuild or repair.
   This maximizes source isolation but can leave rarely updated entities
   unresolvable after eviction or Redis loss.
2. **Always read MongoDB on a miss:** maximizes immediate recovery but can create
   a thundering herd, overload source services, and couple projection freshness
   to every source database.

## Consequences and risks

- Requires the Kafka CDC producer and its event contract to be trustworthy.
- A strict no-fallback policy makes cache completeness a readiness concern.
- Some telemetry proposed for change streams becomes irrelevant or must move to
  the upstream CDC producer.

## Resolved conflicts

- **Resolved by [live ingestion source](01-system-context/live-ingestion-source.md):**
  Kafka topics are the engine input; MongoDB change-stream capture and resume
  tokens belong upstream.
- “Zero cross-database point queries at runtime” versus cache-miss database
  fallbacks in the telemetry draft is resolved as a default-deny rule with
  explicit, bounded relation-level fallback.
- **Resolved by [Redis authority boundary](01-system-context/redis-authority-boundary.md)
  and [durable state ownership](01-system-context/durable-state-ownership.md):**
  Redis is non-authoritative; Kafka owns consumer cursors and the engine-owned
  MongoDB metadata store owns migration, control-plane, and DLQ state.
- **Resolved by [Debezium envelope boundary](02-contracts/cdc-event-envelope.md):**
  the existing upstream envelope is consumed as published; unprocessable events
  go to the durable DLQ before offset advancement.
- **Resolved in principle by [identity and ordering scope](02-contracts/identity-time-ordering.md):**
  Kafka offsets track delivery only; source-scoped metadata fences freshness. The
  exact tuple and comparison scope remain a follow-up decision.
- **Resolved by [transformation execution boundary](02-contracts/transformation-contract.md):**
  Bloblang-in-Go is authoritative; Painless performs only fenced mutations.
- Manual batch offset commits versus committing “remaining” records after
  partial bulk failures without defining partition-contiguous progress remains
  an offset-semantics question; offsets are not freshness revisions.

## Follow-up questions

- Which relation types are allowed to use source-of-truth fallback, and what
  exact rate, timeout, and concurrency budgets do they receive?
- How are fallback reads and failures surfaced in metrics, traces, and
  diagnostics without exposing protected identifiers?

## Validation

- An explicitly permitted cold-cache reference resolves through bounded fallback.
- A non-permitted relation never performs a foreign-database point read.
- Concurrent misses are deduplicated and source load remains within policy.
- Source outages produce bounded pending or repair behavior rather than
  incomplete projections.
- Telemetry distinguishes cache hits, misses, fallback reads, failures, and
  repairs.
