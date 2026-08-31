---
type: Architecture Decision Record
title: "ADR-0018: Reconciliation and repair"
description: Reconciliation compares MongoDB-derived canonical projections with pinned Elasticsearch targets and repairs only through the normal fenced write path.
tags: [architecture, adr, reconciliation, repair, audit, drift]
status: accepted
decision_id: ADR-0018
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Audits pin a physical Elasticsearch target and record a source-scoped boundary or watermark; they never follow a moving alias.
  - Canonical expectations use MongoDB source-of-truth data and the same manifest and transformation rules as normal writes.
  - Repairs are dry-run by default and, when enabled, use the normal fenced, idempotent write path with bounded rate and durable audit history.
  - Redis is not authoritative for reconciliation correctness, and ambiguous or systemic drift requires operator review.
---

# ADR-0018: Reconciliation and repair

## Context

Live at-least-once processing, independent MongoDB sources, Redis-derived
lookups, and blue-green targets can leave a projection temporarily or
permanently divergent from source truth. The design draft proposes sampled
canonical hashing and repair, but direct replacement could bypass source fencing
or mistake secondary lag for a confirmed delete.

## Decision

Reconciliation compares a canonical projection assembled from MongoDB source-of-
truth data with a pinned physical Elasticsearch target. Each run records a
source-scoped boundary or watermark and the exact manifest and transformation
versions used. Redis may be audited separately, but is never authoritative for
the expected document.

Use layered detection: risk-based samples for recently changed,
migration-touched, previously repaired, high-value, or error-prone entities;
random samples for broad coverage; and full scans when validating a migration or
investigating systemic drift. Normalize non-semantic fields with stable
serialization, use a canonical hash as the fast comparison, and retain a
bounded field-level diff for diagnosis. Check missing documents, unexpected
documents after deletion, field mismatches, nested-child membership, and fence
mismatches.

Classify findings as expected transient, deterministic repairable,
infrastructure, or ambiguous/systemic. Recheck transient lag or cross-source
inconsistency after a bounded delay. Repairs are dry-run by default; enabled
repairs are deduplicated, rate-limited, auditable, and pass through the normal
identity, transformation, deletion, fencing, and idempotent write path. Never
replace a document through an unfenced direct index operation.

## Alternatives considered

1. Run only full scans and directly replace mismatching Elasticsearch documents.
   This is straightforward but expensive, unsafe against newer fenced state, and
   unable to prove a globally consistent view across independent sources.
2. Compare against Redis-enriched projections as the expected truth. This is
   cheaper for some lookups but makes cache staleness or loss appear as source
   drift and violates Redis's derived authority boundary.
3. Alert on hash mismatches without field diffs or repair. This minimizes write
   risk but leaves silent data loss dependent on manual investigation and makes
   recurring causes difficult to classify.

## Consequences

- Audit records must retain target, source boundary, manifest/transformation
  versions, sample method, findings, and repair outcomes.
- Sampling detects risk but cannot prove the absence of drift; migration gates
  and investigations may require full scans.
- MongoDB primary or equivalent source evidence is needed before treating a
  missing source document as a confirmed delete when secondary lag is possible.
- Repair traffic needs independent rate limits and observability so it cannot
  starve live ingestion.

## Validation

- Missed, duplicated, stale, and deleted projections are detected against a
  pinned physical target.
- Transient target lag and inconsistent cross-source observations are deferred
  rather than incorrectly repaired.
- Enabled repairs converge through normal fencing and remain safe to replay.
- Migration audits detect target divergence before alias cutover.
- Redis loss or staleness does not change MongoDB-derived reconciliation truth.

## Review triggers

Revisit if sample coverage misses incidents, full scans exceed capacity or
retention windows, source watermarks cannot distinguish transient from durable
drift, or repair traffic impacts live ingestion.

## Related concepts

- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
