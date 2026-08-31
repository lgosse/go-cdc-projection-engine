---
type: Architecture Decision Record
title: "ADR-0006: Relationship expansion scope"
description: V1 supports rooted acyclic expansion graphs while allowing scalar cross-references.
tags: [architecture, adr, relationships, joins, fanout]
status: accepted
decision_id: ADR-0006
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0006: Relationship expansion scope

## Context

The design includes root fields, cached references, nested children, and a
multi-hop dispute relation. It also requires child-before-parent convergence and
no foreign-database point queries at runtime. A general graph model would make
root resolution, cache invalidation, fan-out, and bootstrap traversal
unbounded. At the same time, useful scalar references such as a related task ID
should not be rejected merely because they point to another entity.

## Decision

V1 supports explicitly declared, rooted, acyclic projection graphs with bounded
cardinality and a declared root-resolution strategy. Acyclic applies to
relationships that expand or resolve data into the projection, not to scalar
cross-reference IDs.

Supported relation types are root fields, cached many-to-one references, bounded
one-to-many nested children, and explicitly declared multi-hop relations resolved
through a cache or reverse index.

Every non-root relation declares its source topic and identity, parent/root
resolution strategy, cardinality and target path, update/delete behavior,
maximum expected fan-out, and rebuildability of required caches or reverse
indexes.

Manifest validation rejects cycles in the expansion graph, ambiguous parent
resolution, and unbounded fan-out. A scalar `related_task_id` is allowed because
it does not recursively expand the related entity.

Self-referential or mutually referential expansions are outside v1. A future
decision would need bounded depth, cycle detection, and truncation semantics.

## Boundaries

- Exact high-fan-out reference propagation remains a separate decision.
- This ADR does not decide whether relations are snapshots, owned children, or
  independent entities in every domain.
- Cross-topic event ordering remains outside the relationship contract.

## Alternatives considered

1. Support a general graph with dynamic joins and arbitrary many-to-many
   traversal. This maximizes flexibility but makes cost and convergence
   unpredictable.
2. Support only direct root fields and children. This simplifies runtime work
   but cannot represent the drafted cached-reference and multi-hop examples.

## Consequences

- Manifests have a finite expansion dependency order and can be validated before
  consumption.
- Scalar cross-references remain possible without recursive document expansion.
- Self-referential and mutually referential expanded projections require a
  separate bounded-recursion design.
- Reverse indexes and cache rebuildability are part of relation review.
- High-fan-out reference changes may require deferred rebuilds or another later
  propagation strategy.

## Validation

- Represent the drafted organisation, shifts, attendances, and dispute examples
  without implicit joins.
- Reject cycles, missing root resolution, ambiguous parents, and unbounded
  cardinality/fan-out in manifest fixtures.
- Verify scalar cross-references do not trigger recursive expansion.
- Exercise child-before-parent and multi-hop histories using only declared
  relation mechanisms.
- Demonstrate that required cache and reverse-index mappings are rebuildable.

## Review trigger

Revisit when a required projection needs self-referential expansion, mutually
referential expansion, unbounded many-to-many relationships, or fan-out beyond
the accepted workload envelope.

## Related concepts

- [Relationship model](../02-contracts/relationship-model.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
