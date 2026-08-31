---
type: Architecture Decision Record
title: "ADR-0014: Cache and reverse-lookup semantics"
description: Redis uses relation-specific derived caches, controlled read-through, and generation-based rebuilds.
tags: [architecture, adr, redis, cache, reverse-index, rebuild]
status: accepted
decision_id: ADR-0014
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Reference caches may expire only when their relation declares bounded staleness and controlled read-through or repair.
  - Reverse indexes required for root resolution are retained or proactively rebuilt and do not casually expire.
  - Rebuilds use a generation boundary and atomic reader cutover after source changes are caught up.
  - Redis remains derived; every cache value and reverse index has a durable or reconstructible source.
---

# ADR-0014: Cache and reverse-lookup semantics

## Context

Redis is a non-authoritative operational store. It holds both refreshable
reference values and reverse indexes needed to resolve multi-hop relationships,
but those data types have different correctness and miss behavior. TTL eviction,
Redis loss, and rebuilds must not permanently prevent valid projections or expose
partially rebuilt state.

## Decision

Treat Redis as a derived store with two distinct cache classes.

Reference-value caches, such as organisation metadata keyed by ID, may use TTLs
when the relation declares an acceptable staleness window. An explicitly
permitted miss policy may perform controlled source-of-truth read-through with
per-key deduplication, timeouts, rate limits, version/fence checks, and cache
repopulation. If the source is unavailable, defer or repair rather than publish
an incomplete derived projection.

Reverse indexes, such as `attendance_id -> task_id`, are operational indexes
required for root resolution. Populate them from source events and bootstrap
scans. A live miss defaults to asynchronous rebuild or repair; synchronous source
lookup is allowed only when the relation explicitly declares an indexed,
bounded, protected fallback. Required reverse indexes do not casually expire.

When a rebuild is needed, construct a new cache generation from a source
snapshot, apply changes after the snapshot boundary, validate completeness, and
switch readers to the new generation atomically. Live writes must either update
the rebuilding generation or be replayed into it before cutover.

Each relation declares its freshness or completeness requirement, miss policy,
retention/TTL policy, rebuildability, and source protection limits. Redis loss
must delay processing or trigger rebuild, never remove authoritative Kafka,
MongoDB metadata, deletion fences, or migration state.

## Alternatives considered

1. Use Redis as the universal lookup path and defer every miss. This preserves
   source isolation but makes rarely updated references permanently
   unresolvable after eviction.
2. Query source MongoDB synchronously for every miss. This maximizes immediate
   recoverability but can create cascading load during cache outages and miss
   storms.
3. Rebuild the active cache namespace in place. This avoids a generation
   pointer but exposes partially rebuilt data and races with live updates.

## Consequences

- Rare reference values can recover through bounded read-through without making
  every cache miss a source query.
- Reverse-index correctness depends on retained or proactively rebuilt indexes,
  not arbitrary TTL expiry.
- Redis loss is recoverable through source snapshots and event replay.
- Cache generations, source boundaries, and completeness state require
  observability and operational ownership.
- Relation-specific staleness rules increase manifest and review complexity.

## Validation

- Reference eviction triggers only an approved, deduplicated, rate-limited
  read-through or repair path.
- Reverse-index misses do not create uncontrolled source-query storms.
- Rebuilds never expose a partially populated generation to readers.
- Changes occurring during a snapshot are present before generation cutover.
- Older cache data cannot overwrite a newer fenced value.
- Redis loss does not lose authoritative state or cause unsafe projection writes.

## Review triggers

Revisit if cache rebuild time exceeds the source/change retention window, if
read-through threatens source capacity, if declared staleness is incompatible
with consumer needs, or if reverse-index completeness cannot be measured.

## Related concepts

- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Redis authority boundary](../01-system-context/redis-authority-boundary.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
