---
type: Architecture Decision Record
title: "ADR-0043: Source-draft conflict resolution"
description: Resolves contradictory source-draft assumptions about foreign-database reads on cache misses.
tags: [architecture, adr, conflicts, cache, mongodb]
status: accepted
accepted_on: 2026-08-31
owner: TBD
---

# ADR-0043: Source-draft conflict resolution

## Status

Accepted on 2026-08-31. Owner: TBD.

## Context

The system draft states that runtime relationship lookups use Redis without
cross-database point queries. The observability draft assumes that a cache miss
may trigger a direct read from an external MongoDB service. Earlier decisions
already made MongoDB authoritative and Redis derived, and allowed controlled
fallback for selected relations. This ADR closes the wording conflict without
changing those runtime policies.

## Decision

Interpret “zero cross-database point queries” as a prohibition on an unrestricted
runtime query model. Normal stream processing uses Redis. A cache miss may read
the authoritative MongoDB source synchronously only when an explicit relation
policy permits it.

Permitted fallback requires per-key deduplication, timeout, rate limiting,
source-protection controls, and cache repopulation. Required reverse-index misses
default to asynchronous rebuild, repair, or bounded pending. If the source is
unavailable, the engine does not invent a value or publish an incomplete
projection; it defers, pauses, repairs, or uses the applicable durable custody
path.

The remaining source-draft conflicts are governed by the accepted decisions
linked from the source-conflicts concept. This ADR does not introduce a new
authority, fallback for every miss, or arbitrary foreign-database query API.

## Alternatives considered

1. **Strict zero point reads:** defer every miss until cache rebuild or repair.
   This maximizes source isolation but can leave rarely updated entities
   unresolvable after eviction or Redis loss.
2. **Always read MongoDB on a miss:** maximizes immediate recovery but can create
   a thundering herd, overload source services, and couple freshness to every
   source database.

## Consequences

- Source isolation remains the default runtime behavior.
- Selected reference relations can recover from cold or evicted Redis entries.
- Relation manifests must declare fallback eligibility and protection limits.
- Reverse-index loss remains a rebuild/repair concern, not permission for
  uncontrolled point reads.
- Metrics and diagnostics must distinguish cache misses, fallback reads, and
  repair outcomes.

## Validation

- An explicitly permitted cold-cache reference resolves through bounded fallback.
- A non-permitted relation never performs a foreign-database point read.
- Concurrent misses are deduplicated and source load remains within policy.
- Source outages produce bounded pending or repair behavior rather than
  incomplete projections.
- Telemetry distinguishes cache hits, misses, fallback reads, failures, and
  repairs.

## Review triggers

Revisit if fallback becomes common rather than exceptional, source protection
limits are insufficient, relation policies cannot express required behavior, or
the source services adopt a different lookup contract.

## Related concepts

- [Source draft conflicts](../source-conflicts.md)
- [Objectives and boundaries](../01-system-context/objectives-and-boundaries.md)
- [Data-store responsibilities](../01-system-context/data-store-responsibilities.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
