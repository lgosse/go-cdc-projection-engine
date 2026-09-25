---
type: Architecture Review Topic
title: Bootstrap consistency
description: Defines how a full source scan converges with concurrent CDC traffic.
tags: [lifecycle, bootstrap, mongodb, consistency]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0015
conditions:
  - Each source uses its production connector's MongoDB source.wallTime/cluster-time watermark as a source-scoped bootstrap boundary.
  - The target receives live writes during scanning and replays the boundary overlap before cutover.
  - Delete safety combines source fences, replay, durable deletion custody, and final reconciliation.
  - Child absence is authoritative only after complete relation enumeration at the boundary; unresolved completeness or ordering blocks cutover (ADR-0051).
  - Bootstrap runs and chunks have durable resumable checkpoints in engine-owned MongoDB metadata.
---

# Bootstrap consistency

## Decision

For each source, record the production connector's MongoDB `source.wallTime`
cluster-time watermark as the bootstrap boundary. Keep the boundary source-
scoped; Kafka offsets remain delivery coordinates and are paired with the
watermark only for replay bookkeeping.

Create a new versioned target and register a durable bootstrap run. Begin live
dual-write processing to that target while the snapshot scan runs. Scan each
MongoDB source using stable keyset ranges discovered from sampled split points,
not arithmetic min/max assumptions. Apply the same identity, source-fence,
transformation, deletion, cache, and Elasticsearch write rules as live events.

After scanning, replay or drain the boundary overlap through a current
source-partition high-watermark. Duplicate snapshot/CDC work is expected and
must be harmless. Do not cut over until every source partition, pending repair,
and related cache generation is caught up and validation succeeds.

Delete safety is proven through a documented combination of:

- source-scoped `wallTime` boundaries and freshness fences;
- complete source-bounded relation enumeration for child membership; an absent
  child is omitted from the rebuilt document only after enumeration completes;
- live dual-write plus overlap replay for deletes occurring during the scan;
- durable MongoDB deletion fences and derived hidden Elasticsearch tombstones;
- final source-to-target reconciliation, including records absent from the
  snapshot and lagging-secondary observations.

Do not manufacture a child revision from snapshot absence. Preserve any known
child fence and keep the run pending if available ordering evidence cannot
reject an older overlap event; see [ADR-0051](../decisions/0051-child-rebuild-from-source-snapshots.md).

Persist bootstrap-run state and per-chunk checkpoints in engine-owned MongoDB.
Mark a chunk complete only after its target writes and required cache work
succeed. A crashed or interrupted run resumes unfinished chunks and retains the
same source boundary.

## Pros

- Makes bootstrap correctness independent of job timing.
- Keyset ranges work with ObjectIDs and uneven key distributions.
- Shared fencing prevents old snapshot data overwriting newer changes.

## Cons and risks

- Cross-database snapshots cannot be globally transactional.
- Capturing and replaying a boundary requires CDC connector support.
- Partial relation scans and unresolved ordering cannot establish child absence
  and may delay target cutover.
- Secondary lag can return stale records unless boundary fencing and final
  reconciliation account for it.

## Alternatives considered

1. Pause live consumption during the snapshot. This simplifies coordination but
   creates backlog, increases cutover time, and conflicts with zero-downtime
   operation.
2. Let snapshot and live writes race using timestamps. This cannot safely
   compare independent sources or connector-delayed events.
3. Use a single global snapshot transaction across all MongoDB services. The
   services do not share a transaction boundary, so this is unavailable without
   changing ownership and deployment assumptions.

## Consequences

- Bootstrap and blue/green migration can proceed without stopping live writes.
- Snapshot/CDC overlap and replay are normal and rely on idempotent fenced
  mutations.
- Connector/source watermark support becomes a production prerequisite.
- MongoDB metadata stores run and chunk checkpoints, boundaries, and validation
  evidence.
- Reconciliation remains necessary because independent sources and secondary
  reads cannot provide one globally transactional view.

## Validation

- Events before, during, and after the `source.wallTime` boundary converge in the
  target.
- Deletes during the scan and stale secondary reads cannot leave resurrected
  documents after cutover.
- Complete relation enumeration omits absent children; incomplete scans or
  unresolved ordering keep the rebuild pending and block cutover.
- Duplicate snapshot/CDC processing is harmless under the shared fencing rules.
- A crashed run resumes unfinished chunks without losing the original boundary.
- Live dual-write and overlap replay catch every source partition up to the
  cutover high-watermark.
- Cache generation and index target are complete and mutually compatible before
  alias cutover.

## Review trigger

Revisit if a connector cannot provide a trustworthy cluster-time watermark, if
source/change retention is shorter than bootstrap catch-up, or if reconciliation
cannot prove delete and cross-source convergence.

## Related concepts

- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Blue/green migration](blue-green-migration.md)
- [Reconciliation and repair](reconciliation-and-repair.md)

## Follow-up questions

- How is each connector's `source.wallTime` watermark acquired and paired with
  replay positions in production?
- What exact chunk checkpoint schema and operator resume controls are required?
- What reconciliation evidence is sufficient to declare delete safety at cutover?
