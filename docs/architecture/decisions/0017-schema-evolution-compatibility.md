---
type: Architecture Decision Record
title: "ADR-0017: Schema evolution compatibility"
description: Projection changes use conservative semantic classification, with only allow-listed additive mappings applied in place and meaning-changing changes migrated blue-green.
tags: [architecture, adr, schema, compatibility, elasticsearch, migration]
status: accepted
decision_id: ADR-0017
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Compatibility is classified from normalized mappings, settings, and transformation metadata rather than serialized document order.
  - Only an explicit allow-list of safe additive mapping changes may update an active index in place.
  - Changes that can alter persisted meaning use a new versioned target and the blue-green migration lifecycle.
  - Unknown or ambiguous changes block affected consumption until resolved.
---

# ADR-0017: Schema evolution compatibility

## Context

Projection mappings are explicitly owned by versioned manifests and production
schemas are strict. Elasticsearch permits some mapping additions in place, but
field types, analyzers, nested semantics, and index-creation settings can require
index recreation. A transformation can change indexed meaning even when its
mapping is unchanged. ADR-0016 provides the durable zero-downtime migration and
rollback lifecycle for a new physical target.

## Decision

Classify manifest and projection changes semantically into four classes:

1. **Runtime-only** changes do not alter persisted document meaning or mapping
   and may roll out normally.
2. **Safe additive** changes add only explicitly allow-listed fields with
   compatible types. They may update the active mapping after validation, and
   older documents and consumers must tolerate the field being absent.
3. **Reindex-required** changes use a new manifest/projection version and
   ADR-0016's blue-green migration. This includes field type changes,
   analyzers, normalizers or index-creation settings, `object`/`nested` changes,
   renames, removals, changed null/default/coercion semantics, changed
   transformation meaning, and identity or relationship changes.
4. **Forbidden/blocked** changes include malformed manifests, ambiguous diffs,
   and undeclared dynamic behavior under strict mappings. Affected source
   progress remains blocked until an operator resolves the mismatch.

Compare normalized mappings, settings, and transformation metadata semantically,
not raw YAML or JSON bytes. Pin the exact manifest and transformation versions
to each physical target. Never silently change an existing field's meaning or
remove a consumer-visible field.

## Alternatives considered

1. Require blue-green migration for every persisted change. This is simpler and
   gives the strongest rollback boundary, but makes harmless additive evolution
   slower and more expensive.
2. Apply any Elasticsearch-compatible mapping update in place. This reduces
   migration cost but cannot safely handle analyzer, semantic, consumer, or
   transformation changes.
3. Rely on dynamic mappings and implicit compatibility. This hides contract
   drift and conflicts with ADR-0009's strict production schema.

## Consequences

- Manifest, projection, and transformation versions become migration inputs.
- Startup validation must produce a deterministic compatibility classification
  before source consumption begins.
- Additive updates need an approval policy and consumer tolerance for absent
  fields.
- Renames and removals require an explicit deprecation or migration plan.
- Conservative classification increases blue-green migrations, storage use, and
  operational review.

## Validation

- Identical normalized inputs are classified as unchanged regardless of
  serialization order.
- A safe additive field can be introduced without invalidating older documents
  or consumers.
- Type, analyzer, nested/object, rename/removal, and transformation-semantic
  changes are blocked from in-place mutation and enter blue-green migration.
- Ambiguous or malformed changes block consumption before source progress.
- Each target records the exact manifest and transformation versions used to
  produce it.

## Review triggers

Revisit if consumer compatibility requirements differ from the allow-list, if
Elasticsearch introduces a mapping/settings capability with different update
semantics, or if migration cost threatens capacity or rollout objectives.

## Related concepts

- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
