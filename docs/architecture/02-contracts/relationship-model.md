---
type: Architecture Review Topic
title: Relationship model
description: Defines supported joins and dependency propagation across source entities.
tags: [contracts, relationships, joins, fanout]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0006
---

# Relationship model

## Decision

V1 supports explicitly declared, rooted, acyclic projection graphs with bounded
cardinality and a declared root-resolution strategy. Acyclic applies to
relationships that expand or resolve data into the projection, not to scalar
cross-reference IDs.

## Accepted relationship scope

- Root entity fields.
- Cached many-to-one references.
- Bounded one-to-many nested children.
- Explicitly declared multi-hop relations resolved through a cache or reverse
  index.

Every non-root relation declares its source topic and identity, parent/root
resolution strategy, cardinality and target path, update/delete behavior,
maximum expected fan-out, and rebuildability of required caches or reverse
indexes.

Manifest validation rejects cycles in the expansion graph, ambiguous parent
resolution, and unbounded fan-out. A scalar cross-reference such as
`related_task_id` is allowed because it does not recursively expand the related
entity.

Self-referential or mutually referential expansions are outside v1. Supporting
them requires a separate decision covering bounded depth, cycle detection, and
truncation semantics.

Exact propagation behavior for high-fan-out reference updates remains a
follow-up decision.

## Pros

- Covers the examples without pretending to be a general graph engine.
- Makes reverse-index and root-resolution requirements explicit.
- Allows useful scalar links without recursive projection complexity.
- Validation can reject relationships the runtime cannot update safely.

## Cons and risks

- Reference updates such as organization changes may fan out to many roots.
- Cached reverse maps can be large and expensive to rebuild.
- Limiting expansion shapes may exclude some self-referential projections.
- Bounded recursion, if later added, will require additional state and limits.

## Questions to stamp

- How are reference-field changes propagated to existing root documents?
- What fan-out limit triggers deferred rebuild instead of live updates?
- Are relations snapshots, owned children, or independent entities?

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Identity, time, and ordering](identity-time-ordering.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Deletion and replay](deletion-and-replay.md)
