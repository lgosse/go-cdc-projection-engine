---
type: Glossary
title: Domain language
description: Establishes consistent terms for the architecture review.
tags: [context, glossary]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0039
accepted_on: 2026-08-31
owner: TBD
conditions:
  - The glossary distinguishes business records, Debezium transport, logical projections, physical targets, and engine metadata.
  - Related entity is the neutral general term; child alone does not imply ownership, while owned-child is an explicit lifecycle role.
  - Envelope, event, source record, projection document, logical projection, physical target, alias, and reverse index are not interchangeable terms.
  - Organization-specific vocabulary conflicts remain a follow-up review trigger.
---

# Domain language

## Decision

Use these canonical terms:

| Term | Meaning |
|---|---|
| **Source record** | A business record stored in authoritative domain MongoDB |
| **Source entity** | A source record considered as a logical domain object |
| **CDC envelope** | The immutable upstream Debezium record carried through Kafka |
| **CDC event** | The interpreted source change represented by a CDC envelope |
| **Projection** | A logical denormalized read model defined by a manifest |
| **Projection document** | One logical document produced for a projection root |
| **Root entity** | The source entity whose identity determines the projection document ID |
| **Related entity** | Any other source entity contributing to a projection |
| **Reference relation** | A relation contributing lookup or denormalized fields |
| **Nested member** | A related entity stored as an independently identified nested item |
| **Snapshot relation** | Related data embedded as a non-independently fenced object |
| **Owned child relation** | A separately identified member belonging to one root's projection lifecycle |
| **Independent entity relation** | A separately identified entity with its own lifecycle that may contribute to multiple roots |
| **Logical projection ID** | Stable identity of a projection across physical index versions |
| **Physical target** | A concrete versioned Elasticsearch index receiving writes |
| **Search alias** | Stable read name pointing to one verified physical target |
| **Active write target set** | Physical targets receiving live writes during normal or dual-write operation |
| **Engine metadata** | MongoDB-owned control, checkpoint, fence, DLQ, replay, and migration state |
| **Source boundary** | A source-time or connector watermark defining bootstrap/catch-up scope |
| **Deletion fence** | Durable metadata preventing stale resurrection |
| **Bootstrap** | Building a projection from MongoDB source state plus accepted overlap replay |
| **Reconciliation** | Comparing a canonical MongoDB-derived projection with a physical target |
| **Repair** | An explicit fenced write intended to correct detected drift |
| **Replay** | Reprocessing retained CDC or DLQ records through normal idempotent handling |
| **Migration** | Changing projection targets or semantics through the blue-green lifecycle |

Use **related entity** as the neutral general term. “Child” by itself does not
imply business or lifecycle ownership; only the explicit **owned child
relation** role carries the lifecycle meaning defined by
[ADR-0059](../decisions/0059-relation-lifecycle-classification.md).

Keep these distinctions explicit:

- a CDC envelope is not the source record;
- a projection is not the source entity;
- a logical projection is not a physical Elasticsearch target;
- a search alias is not an index;
- a reverse index is derived operational data, not canonical state; and
- a deletion fence is metadata, not a projected business document.

## Pros

- Prevents business, transport, projection, storage, and control-plane concepts
  from being conflated.
- Gives manifests, tests, diagnostics, metrics, and runbooks stable nouns.
- Keeps relation terminology neutral about ownership across services.

## Cons and risks

- More precise terminology requires consistent adoption across documentation and
  tooling.
- Existing organization vocabulary may conflict with some definitions.
- “Nested member” and “snapshot relation” require explanation for new authors.

## Alternatives considered

1. Keep the source draft's terminology. This minimizes editing but leaves event,
   child, index, and projection ambiguous.
2. Use only business-domain vocabulary. Familiar to source teams but insufficient
   for transport, migration, and control-plane concepts.
3. Use storage-oriented vocabulary everywhere. Precise for implementation but
   obscures business identity and relation meaning.
4. Rename child everywhere to related entity. This reduces ambiguity but loses a
   useful constrained shorthand for nested relation configuration.

## Consequences

- Contracts and operations should use glossary terms consistently.
- Metrics and diagnostics distinguish logical projections, physical targets,
  source records, envelopes, and metadata.
- Relation terminology does not imply ownership absent from the source system.
- Organization-specific vocabulary conflicts remain a review trigger.

## Validation

- The glossary is linked from contract and operational indexes.
- Manifests and diagnostics consistently distinguish logical projections,
  physical targets, source records, envelopes, and metadata.
- Tests use the same terms for event histories and canonical state.
- No document or metric labels a derived store as authoritative.
- “Child” is used only where the manifest relation subtype is intended.

## Review trigger

Revisit if organization vocabulary conflicts with these definitions, terms cause
repeated implementation errors, or public diagnostics require a different
compatibility vocabulary.

## Related concepts

- [Domain language](domain-language.md)
- [Objectives and boundaries](objectives-and-boundaries.md)
- [Data-store responsibilities](data-store-responsibilities.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
