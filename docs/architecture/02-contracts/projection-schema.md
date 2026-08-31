---
type: Architecture Review Topic
title: Projection schema
description: Defines ownership and limits of the Elasticsearch document shape.
tags: [contracts, elasticsearch, schema, mappings]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0009
---

# Projection schema

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

## Pros

- Prevents accidental field-type drift.
- Engine metadata supports deterministic writes and audits.
- Bounded nested data makes capacity risks reviewable.

## Cons and risks

- Strict mappings increase the operational cost of new fields.
- Hidden metadata increases document and update size.
- Elasticsearch nested documents can be expensive even below hard limits.

## Conditions and boundaries

- The exact metadata namespace and derived-copy fields remain a follow-up
  contract detail.
- The public API must omit engine metadata even when an Elasticsearch document
  stores a derived copy.
- Cardinality and document-size thresholds for separate child indices require
  measured capacity evidence.
- Compatibility rules for manifest versions and consumer-visible field changes
  remain part of manifest and migration decisions.

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
  alternative before it creates unsafe document growth.

## Review trigger

Revisit if consumer compatibility requires relaxed mappings, if Elasticsearch
document or nested-query limits are approached, or if local fencing cannot be
implemented safely with a derived metadata copy.

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Relationship model](relationship-model.md)
- [Identity, time, and ordering](identity-time-ordering.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Transformation contract](transformation-contract.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Blue/green migration](../04-data-lifecycle/blue-green-migration.md)

## Follow-up questions

- What exact metadata namespace and derived-copy fields are required for local
  fencing?
- When should large child sets become separate indices instead of nested arrays?
