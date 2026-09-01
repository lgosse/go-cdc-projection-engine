---
type: Architecture Review Topic
title: Objectives and boundaries
description: Defines what the projection engine owns and deliberately does not own.
tags: [context, scope, boundaries]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0037
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Worker processes are replaceable and do not rely on local durable state; durable control and custody state remains in engine-owned MongoDB and Kafka offsets.
  - The engine consumes existing Debezium CDC from Kafka and does not own CDC capture, source writes, business workflows, public query APIs, or Elasticsearch cluster administration.
  - Controlled source fallback for relation resolution is available only for explicitly allow-listed relation policies.
  - The first production release validates one representative projection; expansion to more projections is evidence-gated.
  - Engine-wide guarantees are enforced by runtime contracts; projection semantics are declared and governed through manifests.
---

# Objectives and boundaries

## Decision

The engine owns:

- consuming existing Debezium CDC envelopes from Kafka;
- validating, ordering, fencing, coalescing, transforming, and routing events;
- reading MongoDB source state for bootstrap, reconciliation, and repair;
- resolving declared relationships through Redis policies and controlled,
  allow-listed fallback;
- maintaining versioned Elasticsearch projections and aliases;
- maintaining engine-owned MongoDB control, checkpoint, fence, DLQ, and
  migration state;
- exposing health, diagnostics, telemetry, and operator workflows; and
- coordinating zero-downtime bootstrap and blue-green migrations.

The engine does not own source database writes or business workflows, CDC
capture, Debezium operation or Kafka publication, public search APIs or query
authorization, Elasticsearch cluster administration, domain schema ownership,
exactly-once processing, cross-database transactions, arbitrary ETL, unlimited
relationship recursion, unrestricted dynamic mappings, or direct MongoDB
change-stream consumption in stream mode.

“Stateless” means worker processes are replaceable and do not rely on local
durable state. The overall system still has durable Kafka offsets and
engine-owned MongoDB control and custody state.

Normal stream processing avoids direct foreign-database point reads. A
relation-specific cache miss may use controlled source fallback only when the
relation policy explicitly allows it; this is not a general runtime query model.

Source services provide authoritative business data and CDC. The back-end team
owns manifests, projection semantics, and engine behavior; the lead owns
high-risk architecture and lifecycle decisions; DevOps owns infrastructure and
deployment primitives; query clients consume stable search aliases; and the DPO
governs privacy and retention interpretation.

The first production release validates one representative projection containing
a root entity, a reference lookup, a nested child, an out-of-order or
missing-parent case, and the complete stream, bootstrap, migration, replay,
repair, and reconciliation lifecycle. The engine contract remains domain
agnostic, but additional projections are added only after capacity, correctness,
and operational evidence supports them.

Engine-wide guarantees are enforced by runtime contracts and accepted
correctness rules. Projection-specific meaning, mappings, relationships, and
transformations are declared and governed through versioned manifests.

## Pros

- Establishes a reusable engine instead of a domain-specific indexer.
- Keeps source ownership with existing services and query ownership with clients.
- Makes projections disposable and rebuildable.
- Corrects the misleading implication that no durable state exists.
- Makes first-release scope measurable while preserving multi-projection design.

## Cons and risks

- End-to-end correctness depends on upstream CDC contracts outside the engine.
- A generic engine can still push domain complexity into manifests and scripts.
- Allow-listed source fallback adds a bounded runtime dependency on MongoDB.
- A single representative first release delays broad projection coverage.
- Expansion requires repeated capacity and operational validation.

## Alternatives considered

1. Own CDC capture and public query APIs as well as projection. This simplifies
   nominal ownership but couples the engine to existing systems and workflows.
2. Treat the engine as a stateless writer with no durable control state. This
   cannot safely support DLQ custody, migrations, checkpoints, or recovery.
3. Forbid all source fallback on cache misses. This preserves a strict no-point-
   read rule but makes cold starts and required relationship resolution block
   unnecessarily.
4. Commit to all source-draft projections immediately. This makes capacity,
   correctness, and operational readiness assumptions untestable.

## Consequences

- Scope, non-goals, and ownership must be documented alongside the engine.
- Upstream CDC contracts and source retention remain external dependencies.
- Fallback-capable relations need explicit manifest policy and observability.
- The first production release has a deliberately bounded projection scope.
- Adding projections becomes a measured release decision rather than an implicit
  promise.

## Validation

- A scope document lists owned capabilities and explicit non-goals.
- One representative projection exercises stream, bootstrap, migration, replay,
  repair, and reconciliation end to end.
- No engine component writes source business data or owns CDC capture.
- Controlled fallback cannot become unrestricted cross-database querying.
- Worker restart or replacement does not lose durable control or custody state.
- Capacity and release evidence support each subsequent projection addition.

## Review trigger

Revisit if the engine must own another lifecycle boundary, source fallback cannot
remain allow-listed, public query responsibilities move into the service, or
projection expansion repeatedly fails capacity or correctness evidence.

## Related concepts

- [Objectives and boundaries](objectives-and-boundaries.md)
- [Live ingestion source](live-ingestion-source.md)
- [Data-store responsibilities](data-store-responsibilities.md)
- [System guarantees](system-guarantees.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Ownership and governance](../07-operations/ownership-and-governance.md)
