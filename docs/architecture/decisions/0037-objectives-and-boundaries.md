---
type: Architecture Decision Record
title: "ADR-0037: Objectives and boundaries"
description: The engine owns Kafka-driven projection processing and lifecycle coordination while source capture, business workflows, query APIs, and cluster administration remain external.
tags: [architecture, adr, context, scope, boundaries]
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

# ADR-0037: Objectives and boundaries

## Context

The source draft describes a stateless, configuration-driven engine that
consumes CDC from Kafka, reads MongoDB and Redis, and writes Elasticsearch
projections. Existing services already own source data and CDC publication, and
query clients consume published aliases. Accepted decisions add durable
engine-owned MongoDB state for control, custody, fences, and migrations.

## Decision

The engine owns consuming existing Debezium CDC from Kafka; validating,
ordering, fencing, coalescing, transforming, and routing events; reading
MongoDB for bootstrap, reconciliation, and repair; resolving declared
relationships through Redis and explicitly allow-listed fallback; maintaining
versioned Elasticsearch projections and aliases; maintaining engine-owned
MongoDB control, checkpoint, fence, DLQ, and migration state; exposing health,
diagnostics, telemetry, and operator workflows; and coordinating zero-downtime
bootstrap and blue-green migrations.

The engine does not own source database writes or business workflows, CDC
capture, Debezium operation or Kafka publication, public search APIs or query
authorization, Elasticsearch cluster administration, domain schema ownership,
exactly-once processing, cross-database transactions, arbitrary ETL, unlimited
relationship recursion, unrestricted dynamic mappings, or direct MongoDB
change-stream consumption in stream mode.

“Stateless” applies to replaceable worker processes, not to the overall system:
Kafka offsets and engine-owned MongoDB control and custody state remain durable.
Normal stream processing avoids direct foreign-database point reads. A
relation-specific cache miss may use controlled source fallback only when its
manifest relation policy explicitly allows it.

The first production release validates one representative projection with a root
entity, reference lookup, nested child, out-of-order or missing-parent case, and
the complete stream, bootstrap, migration, replay, repair, and reconciliation
lifecycle. The engine remains domain agnostic, but each additional projection is
added only after capacity, correctness, and operational evidence supports it.

Engine-wide guarantees are runtime contracts. Projection-specific meaning,
mappings, relationships, and transformations are versioned manifest concerns.

## Alternatives considered

1. **Own CDC capture and public query APIs.** This simplifies nominal ownership
   but couples the engine to existing systems and workflows.
2. **Treat the engine as a stateless writer with no durable control state.** This
   cannot safely support DLQ custody, migrations, checkpoints, or recovery.
3. **Forbid all source fallback on cache misses.** This preserves a strict
   no-point-read rule but unnecessarily blocks cold starts and required relation
   resolution.
4. **Commit to all source-draft projections immediately.** This makes capacity,
   correctness, and operational readiness assumptions untestable.

## Consequences

- Scope, non-goals, and ownership are documented alongside the engine.
- Upstream CDC contracts and source retention remain external dependencies.
- Fallback-capable relations need explicit manifest policy and observability.
- The first production release has deliberately bounded projection scope.
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

## Review triggers

Revisit if the engine must own another lifecycle boundary, source fallback cannot
remain allow-listed, public query responsibilities move into the service, or
projection expansion repeatedly fails capacity or correctness evidence.

## Related concepts

- [Objectives and boundaries](../01-system-context/objectives-and-boundaries.md)
- [Live ingestion source](../01-system-context/live-ingestion-source.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [System guarantees](../01-system-context/system-guarantees.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Ownership and governance](../07-operations/ownership-and-governance.md)
