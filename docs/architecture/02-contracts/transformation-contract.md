---
type: Architecture Review Topic
title: Transformation contract
description: Defines supported derivations and consistent execution across stream and batch modes.
tags: [contracts, transformations, bloblang, correctness]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0007
conditions:
  - Bloblang execution is deterministic and resource-bounded.
  - Incomplete context is deferred or repaired unless explicitly allowed by the manifest.
  - Malformed or impossible events follow the durable DLQ policy.
  - Each physical target pins the manifest hash and transformation execution identity that produced its documents; semantic changes use a new target.
---

# Transformation contract

## Decision

Bloblang in Go is the authoritative transformation runtime. Elasticsearch
Painless applies only deterministic, idempotent, source-fenced mutations and
does not implement a second business-transformation language.

## Accepted boundary

- Compile a restricted, resource-bounded Bloblang subset before consumption.
- Use the same evaluator in stream, bootstrap, audit, repair, and migration
  modes.
- Evaluate against a canonical assembled projection representation.
- Use the deterministic, versioned operation allowlist and measured resource
  budget dimensions defined by
  [ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md).
- Bind each physical target to an immutable manifest hash, transformation
  version, and a stable digest of the canonical normalized transformation
  definition plus its versioned allowlist identity. Keep this pin in
  authoritative engine-owned target metadata and expose it in diagnostics; a
  per-document public version field is not required. A transformation version
  paired with a different digest is invalid and blocks preflight before source
  progress ([ADR-0062](../decisions/0062-transformation-version-target-association.md)).
- Treat a change that can alter persisted output meaning as reindex-required.
  Create a new physical target and use the existing blue-green lifecycle. While
  both targets are active, evaluate each target's own pinned transformation
  identity; bootstrap and verify the new target before alias cutover. A runtime
  or evaluator upgrade that preserves semantics follows the compatibility
  contract; one that can change output meaning follows this migration rule.
- Do not publish derived values from incomplete context unless the manifest
  explicitly declares the derivation safe with missing fields.
- Defer or repair valid events that lack enough context; classify malformed or
  impossible events through the accepted durable-DLQ policy.
- Keep numeric transformation, event, and document limits evidence-gated under
  Q-070; keep exact function names tied to the pinned evaluator and versioned
  allowlist under ADR-0060 and the target identity under ADR-0062.
- Apply v1 source-entity/relation-level invalidation to participating
  create/update events, owned-child deletes, and independent-reference deletes
  according to the manifest's materialized dependency graph. Reference deletion
  removes optional derived fields or follows an explicitly safe missing-value
  rule ([ADR-0061](../decisions/0061-projection-dependency-invalidation.md),
  [ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).

## Pros

- Stream, bootstrap, audit, repair, and migration share one transformation
  implementation.
- Avoids a difficult and potentially incomplete Bloblang-to-Painless compiler.
- Startup compilation catches syntax errors before consumption.
- Keeps Elasticsearch focused on fenced mutation mechanics.

## Cons and risks

- A partial child event may not contain enough context to re-evaluate a whole
  projection and must be deferred or repaired.
- Complex scripts can undermine manifest safety and performance.
- Canonical assembly in stream mode may require more cached state than drafted.
- Resource limits and pending work add operational states to observe and own.

## Questions to stamp

- **Resolved by [ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md):** What exact Bloblang operations and resource limits are allowed? Use a deterministic data-only allowlist over the canonical projection; measure complexity, byte size, collection work, evaluation cost, and worker resources, with numeric caps set by Q-070.
- **Resolved by [ADR-0061](../decisions/0061-projection-dependency-invalidation.md):** Which dependency changes trigger recomputation? Every accepted change to an entity participating in the declared materialized dependency graph recomputes its dependent roots, even when the changed field is unused; incomplete reverse indexes must be repaired before publishing.
- **Resolved by [ADR-0062](../decisions/0062-transformation-version-target-association.md):** How are transformation versions associated with projected documents? Pin the manifest hash, transformation version, and normalized transformation/allowlist digest in authoritative metadata per physical target; semantic changes use a new blue-green target, without a public per-document version marker.

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Relationship model](relationship-model.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
