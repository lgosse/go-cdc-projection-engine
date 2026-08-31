---
type: Architecture Decision Record
title: "ADR-0010: Elasticsearch write semantics"
description: Elasticsearch mutations are versioned, source-fenced, idempotent bulk operations with explicit dual-write completion.
tags: [architecture, adr, elasticsearch, writes, idempotency, bulk]
status: accepted
decision_id: ADR-0010
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Every pinned active write target must succeed or receive an explicitly terminal disposition before source progress advances.
  - Event-specific permanent Elasticsearch failures go to the durable DLQ; infrastructure-wide failures remain retryable and block progress as required.
  - MongoDB remains authoritative for fences; any Elasticsearch tombstone or metadata is derived and rebuildable.
  - Painless scripts are generated and versioned from validated manifests; Bloblang remains the business-transformation authority.
---

# ADR-0010: Elasticsearch write semantics

## Context

The projection schema requires explicit versioned mappings, while identity,
ordering, transformation, and deletion decisions require source-fenced,
idempotent updates. Kafka delivery is at-least-once and blue/green migration may
require several active Elasticsearch targets at once. Elasticsearch bulk
responses are itemized, so a request can partially succeed.

## Decision

Use bulk scripted upserts keyed by logical projection document ID. Scripts apply
already transformed canonical data and perform only deterministic, idempotent
mutations: root updates, nested-child changes, source-fence comparison, and
derived local metadata updates.

Coalesce events by projection document and pin the active write-target set at
batch start. Send the same logical mutation to every pinned target and inspect
each bulk item independently. Source progress advances only after every required
target succeeds or the event receives an explicitly terminal disposition.

If one target succeeds and another fails, keep the event incomplete and retry the
failed target. Replaying the successful target must be safe and produce no
harmful duplicate effect.

Classify event-specific permanent failures, such as mapping, script, or document
validation errors, through the durable DLQ policy. Treat transport, throttling,
cluster, and target-availability failures as retryable or progress-blocking
infrastructure failures rather than creating one DLQ record per affected event.

Generate versioned Painless mutation templates from validated manifests and load
them at startup. Bloblang remains the business-transformation authority;
Painless does not reimplement business derivations. `retry_on_conflict` may
assist with Elasticsearch races but never replaces source fencing or
idempotency.

For root deletion, Elasticsearch may retain a hidden, non-searchable derived
tombstone or fence marker for the fencing horizon. MongoDB remains authoritative
for the deletion fence, and the derived marker must be rebuildable.

## Alternatives considered

1. Replace the whole projection document after reading or assembling the current
   state. This simplifies shape reasoning but introduces read-before-write races,
   larger payloads, and unsafe stale nested updates.
2. Consult MongoDB for authoritative fencing on every Elasticsearch mutation.
   This avoids a derived Elasticsearch marker but violates the runtime goal of
   avoiding cross-database point queries and adds latency and failure coupling.
3. Store scripts in Elasticsearch and reference them by name. This reduces
   request size but introduces a separate script registry and deployment
   synchronization surface.

## Consequences

- Duplicate and reordered deliveries can converge without read-before-write
  coordination.
- A blocked or unavailable dual-write target can hold source progress until it
  recovers or an operator records a terminal disposition.
- Event-specific permanent failures are inspectable and replayable through the
  durable DLQ, while broad infrastructure failures are not misclassified as
  bad data.
- Generated scripts are self-contained and versioned with manifests, but script
  payloads and startup validation become operational concerns.
- Hidden Elasticsearch tombstones support local delete fencing without making
  Elasticsearch authoritative.

## Validation

- Duplicate and reordered root and child events produce one converged result.
- A partial dual-write succeeds after retry without losing source progress or
  duplicating side effects.
- Every bulk item receives an explicit success, retry, or terminal disposition.
- Mapping, script, and validation failures reach durable DLQ custody; cluster or
  transport failures remain retryable or block progress.
- Script versions are validated against manifest and index versions before
  consumption starts.
- A stale update cannot recreate a root after deletion while the derived marker
  is retained, and MongoDB remains sufficient to rebuild that marker.

## Review triggers

Revisit if dual-write latency or failure rates make partition progress unsafe,
if generated scripts exceed request or compilation budgets, or if the runtime
must support a sink without equivalent fenced mutation semantics.

## Related concepts

- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Blue/green migration](../04-data-lifecycle/blue-green-migration.md)
