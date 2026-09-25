---
type: Architecture Decision Record
title: "ADR-0058: Reference fan-out execution threshold"
description: Defines when reference-change propagation uses live updates or deferred recomputation.
tags: [architecture, adr, relationships, fanout, capacity]
status: accepted
decision_id: ADR-0058
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Each materialized reference relation uses a live-update ceiling established by representative benchmarks.
  - Fan-outs at or below the ceiling use bounded live recomputation; larger valid fan-outs become durable deferred recomputation work.
  - The author-declared expected fan-out remains a planning and alert value, not the measured live-update ceiling.
  - Numeric ceilings are recorded as production capacity evidence under Q-070 before production use.
  - The separate hard safety envelope and its smallest-safe-scope blocking behavior remain governed by ADR-0056.
---

# ADR-0058: Reference fan-out execution threshold

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

Under [ADR-0057](0057-reference-change-propagation.md), a mutable referenced
entity change schedules recomputation for every dependent root found through
the reverse index. A small fan-out can be handled as live work. A large fan-out
can consume capacity needed by unrelated stream processing if the engine tries
to apply every root update inline.

The relationship manifest already declares expected fan-out, while
[ADR-0056](0056-nested-relation-capacity-boundaries.md) separates author
expectations from engine-enforced hard safety limits. These values serve
different purposes and must remain distinct.

## Decision

Set a measured live-update ceiling for each materialized reference relation,
using representative benchmarks for the projection and target workload. When a
reference change affects no more than the relation's ceiling, process its root
recomputations through bounded live batches. When it affects more roots, retain
the valid change as durable deferred recomputation work and let workers process
the affected roots in bounded batches.

Crossing the live-update ceiling changes the processing path; it does not make
the source change invalid and does not send it to the DLQ. The projection
author's expected fan-out remains a planning and alert value, not a substitute
for this measured threshold. Crossing the separate engine hard safety envelope
continues to block the smallest safe scope and durably preserve valid work under
ADR-0056.

The numerical ceilings are not set by this ADR. Record them with representative
benchmark evidence under Q-070 before enabling production relations that use
materialized reference propagation. Values used in synthetic proving work are
test inputs, not production capacity claims.

## Alternatives considered

1. **Always process every affected root as live work.** This avoids a second
   execution path, but a large reference fan-out can create long delays and
   starve unrelated projections.
2. **Use one global count for every relation.** This is easy to configure but
   ignores differences in document size, target cost, and update work.
3. **Always defer reference changes.** This gives one predictable path but
   adds unnecessary staleness and operational backlog for small fan-outs.

## Consequences

- Each relation needs a measured live-update ceiling and evidence that the
  target environment can sustain it.
- Larger fan-outs may leave affected roots showing their prior values until the
  deferred work catches up; progress and backlog need operational visibility.
- Deferred recomputation must be durable, bounded, restartable, and fair to
  unrelated work.
- Expected fan-out warnings and engine hard safety limits remain separate from
  the live-versus-deferred execution threshold.

## Validation

- Benchmark representative small and large reference fan-outs against live
  freshness, target capacity, and unrelated-work progress objectives.
- Verify fan-outs at or below the measured ceiling complete through bounded
  live work.
- Verify fan-outs above the ceiling become durable deferred work, survive
  interruption, and eventually recompute every affected root.
- Verify a large but valid fan-out is not DLQed and cannot starve unrelated
  projections.
- Verify crossing the separate hard safety envelope follows ADR-0056.

## Review triggers

Revisit the thresholds if relation shape, document size, target performance, or
workload distribution changes materially; if deferred work cannot catch up; or
if live propagation violates freshness or fairness objectives.

## Related concepts

- [Relationship model](../02-contracts/relationship-model.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Nested relation capacity boundaries](0056-nested-relation-capacity-boundaries.md)
- [Reference change propagation](0057-reference-change-propagation.md)
- [Follow-up register](../follow-ups.md) (Q-014, Q-070)
