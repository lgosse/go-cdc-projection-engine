---
type: Architecture Decision Record
title: "ADR-0057: Reference change propagation"
description: Defines how changes to mutable referenced entities refresh existing root projections.
tags: [architecture, adr, relationships, references, projections]
status: accepted
decision_id: ADR-0057
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Referenced-entity changes use a maintained reverse index to identify affected roots and create durable recomputation work.
  - Recomputations run in bounded batches and apply relation-derived values under contributor-scoped source revision fences.
  - A missing or incomplete reverse index must be repaired or rebuilt before its results are used to publish projection data.
  - The fan-out threshold for switching from live updates to deferred rebuild remains open under Q-014.
---

# ADR-0057: Reference change propagation

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

V1 permits cached references and multi-hop relations that materialize referenced
fields into root Elasticsearch documents. A change to the referenced entity can
therefore leave existing roots stale even after the reference cache is updated.
The relationship model already requires reverse indexes needed for root
resolution to be retained or proactively rebuilt, and ADR-0055 defines
contributor-scoped fences for derived Elasticsearch writes.

## Decision

For a mutable referenced-entity change, use the maintained reverse index to
identify root projection entities that depend on the changed entity. Record
durable recomputation work for those roots and process it in bounded batches.
Recompute from the current root and relation state, then apply relation-derived
values under the contributor-scoped source revision fence so an older reference
event cannot overwrite a newer value.

If the reverse index is missing or its completeness is uncertain, repair or
rebuild it before publishing based on its results. A missing index entry is not
evidence that no roots depend on the referenced entity. This decision does not
set a numeric fan-out threshold or the policy for switching to deferred
rebuild; Q-014 owns that choice. Root events that change their own relation keys
continue through normal root recomputation and relation-index maintenance.

## Alternatives considered

1. **Update only the reference cache.** This makes future projections see the
   new value but leaves existing root documents stale.
2. **Scan all root documents after every reference change.** This avoids a
   reverse-index dependency but makes each reference mutation potentially
   unbounded in cost and complicates completeness during concurrent updates.
3. **Store only the reference ID and resolve it at read time.** This avoids
   propagation work, but requires consumers to make another lookup and may
   change search and read semantics.

## Consequences

- Reverse-index completeness, repair, and rebuild become correctness concerns
  for materialized references.
- Asynchronous recomputation means affected roots may show the prior value
  until their durable work completes; the fan-out limit and rebuild handoff are
  decided separately in Q-014.
- Contributor-scoped fencing prevents an older reference revision from
  overwriting newer relation-derived state without claiming a total order across
  independent source entities.
- Reference fields remain convenient to search in Elasticsearch, at the cost
  of maintaining indexes and scheduling recomputations.

## Validation

- A reference update recomputes every indexed dependent root and leaves
  unrelated roots unchanged.
- Duplicate and stale reference events cannot regress a root's relation-derived
  values.
- Missing or incomplete reverse-index state enters repair or rebuild before
  publication; it is never treated as an empty dependency set.
- Recomputations survive process interruption and make progress in bounded
  batches.

## Review triggers

Revisit if reverse-index repair cannot establish completeness, if asynchronous
staleness exceeds consumer needs, or if measured fan-out makes the selected
live-update and rebuild policy unsafe or too costly; its benchmark-derived
threshold is defined by [ADR-0058](0058-reference-fanout-execution-threshold.md).

## Related concepts

- [Relationship model](../02-contracts/relationship-model.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Derived Elasticsearch fence metadata](0055-derived-elasticsearch-fence-metadata.md)
- [Follow-up register](../follow-ups.md) (Q-013, Q-014)
