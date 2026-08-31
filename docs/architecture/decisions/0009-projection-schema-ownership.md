---
type: Architecture Decision Record
title: "ADR-0009: Projection schema ownership"
description: Manifests own explicit versioned Elasticsearch projections while MongoDB remains authoritative for engine metadata.
tags: [architecture, adr, elasticsearch, schema, mappings]
status: accepted
decision_id: ADR-0009
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Production mappings reject undeclared fields unless dynamic behavior is explicitly enabled by a manifest.
  - MongoDB remains authoritative for source fences and engine control state; Elasticsearch metadata is derived only.
  - Nested relations have declared cardinality and high-cardinality alternatives are reviewed separately.
---

# ADR-0009: Projection schema ownership

## Context

The design draft makes each projection manifest declare Elasticsearch mappings,
settings, aliases, and nested relationship fields. The accepted relationship,
transformation, and deletion decisions require stable field paths, child
identity, and fencing behavior. The accepted state-ownership decision also makes
MongoDB metadata authoritative while Elasticsearch remains rebuildable.

## Decision

The manifest owns the logical projection schema and explicit Elasticsearch
mapping/settings for each schema version. Physical indices are versioned and
read/write aliases control which version is active.

Production mappings default to rejecting undeclared fields. Dynamic mapping
requires an explicit manifest opt-in.

Root fields and projected relationship fields form the consumer-visible
projection. Engine metadata is never part of the public API contract. MongoDB
metadata remains authoritative for source fences, reconciliation state, and
other engine control data; Elasticsearch may contain a derived metadata copy
when a local scripted mutation needs it, but that copy is rebuildable and never
the sole authority.

Independent child entities with their own identity and fencing use bounded
Elasticsearch `nested` arrays. Single embedded snapshots remain ordinary
embedded objects unless query requirements later justify nested semantics.
Every nested relation declares a maximum expected cardinality. High-cardinality
relations require a separate follow-up design rather than silently becoming
unbounded nested arrays.

## Conditions and boundaries

- The exact metadata namespace and derived-copy fields remain a follow-up
  contract detail.
- Public APIs must omit engine metadata even when an Elasticsearch document
  stores a derived copy.
- Cardinality and document-size thresholds for separate child indices require
  measured capacity evidence.
- Compatibility rules for manifest versions and consumer-visible field changes
  remain part of manifest and migration decisions.

## Alternatives considered

1. Allow dynamic mappings and store engine metadata alongside business fields.
   This reduces manifest maintenance but permits accidental type drift, makes
   schema changes harder to review, and blurs the public/internal boundary.
2. Keep all fencing and reconciliation state only in Elasticsearch. This makes
   local scripts simpler but loses correctness state when a projection is
   rebuilt and contradicts the accepted MongoDB authority model.
3. Flatten every relationship into ordinary object arrays. This avoids nested
   mapping cost but can cross-correlate fields from different child members and
   cannot safely represent independently fenced children.

## Consequences

- Mapping changes require explicit manifest review and usually a versioned
  index migration.
- Elasticsearch scripts may use derived metadata for atomic local checks, but
  recovery and reconciliation must consult authoritative MongoDB metadata.
- Nested child queries retain member-level correlation at additional storage
  and update cost.
- High-cardinality relationships cannot be accepted without capacity evidence
  and may require separate indices or projections.
- Consumers receive a stable business projection rather than engine control
  fields.

## Validation

- The same manifest produces compatible mappings in live, bootstrap, and
  migration modes.
- Undeclared fields fail validation or ingestion unless dynamic behavior was
  explicitly opted into.
- Nested child queries preserve per-member field correlation.
- Elasticsearch metadata loss does not remove authoritative MongoDB fencing or
  reconciliation state.
- Public responses and documented consumer fields exclude engine metadata.
- A high-cardinality relation is rejected or routed to an explicitly reviewed
  alternative before unsafe document growth occurs.

## Review triggers

Revisit if consumer compatibility requires relaxed mappings, if Elasticsearch
document or nested-query limits are approached, or if local fencing cannot be
implemented safely with a derived metadata copy.

## Related concepts

- [Projection schema](../02-contracts/projection-schema.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Blue/green migration](../04-data-lifecycle/blue-green-migration.md)
