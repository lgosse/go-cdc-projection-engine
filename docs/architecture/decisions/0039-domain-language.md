---
type: Architecture Decision Record
title: "ADR-0039: Domain language"
description: A canonical glossary distinguishes business records, CDC transport, logical projections, physical targets, and engine metadata.
tags: [architecture, adr, context, glossary, terminology]
status: accepted
decision_id: ADR-0039
accepted_on: 2026-08-31
owner: TBD
conditions:
  - The glossary distinguishes business records, Debezium transport, logical projections, physical targets, and engine metadata.
  - Related entity is the neutral general term; child is retained only as a constrained manifest relation subtype.
  - Envelope, event, source record, projection document, logical projection, physical target, alias, and reverse index are not interchangeable terms.
  - Organization-specific vocabulary conflicts remain a follow-up review trigger.
---

# ADR-0039: Domain language

## Context

The source drafts and architecture concepts use “entity,” “event,” “projection,”
“root,” “child,” “index,” and “alias” across business, transport, storage, and
control-plane contexts. Without stable definitions, later contracts and tests
can accidentally treat a Kafka envelope as canonical data or an Elasticsearch
index as the logical projection.

## Decision

Adopt a canonical glossary distinguishing source records and entities, immutable
Debezium CDC envelopes and interpreted CDC events, logical projections and
projection documents, root and related entities, reference relations, nested
members and snapshot relations, logical projection IDs, physical Elasticsearch
targets, search aliases, active write target sets, engine metadata, source
boundaries, deletion fences, bootstrap, reconciliation, repair, replay, and
migration.

Use **related entity** as the neutral general term. “Child” remains only as a
constrained manifest relation subtype and does not imply business or lifecycle
ownership.

Keep explicit distinctions: an envelope is not a source record; a projection is
not a source entity; a logical projection is not a physical target; an alias is
not an index; a reverse index is derived operational data; and a deletion fence
is metadata rather than a projected business document.

## Alternatives considered

1. **Keep source-draft terminology.** Minimal editing but leaves event, child,
   index, and projection ambiguous.
2. **Use only business-domain vocabulary.** Familiar to source teams but
   insufficient for transport, migration, and control-plane concepts.
3. **Use storage-oriented vocabulary everywhere.** Precise for implementation
   but obscures business identity and relation meaning.
4. **Rename child everywhere to related entity.** Reduces ambiguity but removes a
   useful constrained shorthand for nested relation configuration.

## Consequences

- Contracts, manifests, tests, metrics, diagnostics, and runbooks should use the
  glossary consistently.
- Logical projections, physical targets, source records, envelopes, and metadata
  are separately represented in operational vocabulary.
- Relation terminology does not imply ownership absent from the source system.
- Organization-specific vocabulary conflicts remain a review trigger.

## Validation

- The glossary is linked from contract and operational indexes.
- Manifests and diagnostics consistently distinguish logical projections,
  physical targets, source records, envelopes, and metadata.
- Tests use the same terms for event histories and canonical state.
- No document or metric labels a derived store as authoritative.
- “Child” is used only for the intended manifest relation subtype.

## Review triggers

Revisit if organization vocabulary conflicts with these definitions, terms cause
repeated implementation errors, or public diagnostics require a different
compatibility vocabulary.

## Related concepts

- [Domain language](../01-system-context/domain-language.md)
- [Objectives and boundaries](../01-system-context/objectives-and-boundaries.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
