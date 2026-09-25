---
type: Architecture Review Topic
title: Blue-green migration
description: Defines a recoverable state machine for provisioning, dual-write, validation, and cutover.
tags: [lifecycle, migration, dual-write, elasticsearch]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0016
conditions:
  - Migration state, leases, fencing tokens, target sets, and operator actions are durable in engine-owned MongoDB.
  - Live dual-write starts before bootstrap and remains active through the rollback window.
  - Cutover requires explicit verification gates and authorized approval.
  - Old targets are never retired automatically and remain protected by retention and rollback checks.
---

# Blue-green migration

## Decision

Model migration as a durable, idempotent state machine per logical projection:

```text
planned -> target provisioned -> dual-write active -> bootstrap running
  -> catch-up complete -> verification passed -> alias cut over
  -> rollback window -> old target quiesced -> retirement approved -> retired
```

Store migration state, active target set, lease, fencing token, checkpoints, and
operator actions in engine-owned MongoDB. Redis may mirror routing data but is not
authoritative. Only one migration may control a logical projection/version at a
time, and expired leases prevent further state mutation by the old owner.

Provision and validate the new versioned target with its immutable manifest hash,
transformation version, and transformation digest. Each active physical target
uses its own pinned transformation identity during live dual-write; never write
one target's transformed output into a target pinned to different semantics.
Activate dual-write before bootstrap and use the accepted source-boundary and
overlap-replay procedure. Cut over the read alias only after mapping, target
identity, bootstrap, source high-watermark, cache-generation, delete/fence, and
sampled canonical-consistency checks pass.

Keep the old target in dual-write during a configured rollback window. If the new
target fails, atomically move the read alias back while the old target remains
current. If old-target dual-write becomes unavailable, mark the migration
degraded and block retirement; rollback is not claimed unless the old target is
kept current or its missed interval is replayed.

Retire the old target only after the rollback window, verification evidence,
backup/rebuild evidence, alias checks, and explicit authorized approval. Never
delete it automatically at cutover.

## Pros

- Supports retry and operator inspection after partial failures.
- Keeps rollback available through a retention window.
- Explicit worker acknowledgment closes the race before bootstrap.

## Cons and risks

- Adds a control-plane component or stricter deployment orchestration.
- Long dual-write windows increase load and divergence surface.
- Rollback after new-only writes may require reverse migration.
- A failed old-target write during the rollback window can remove the rollback
  guarantee unless it is repaired or replayed.

## Alternatives considered

1. Update mappings in place whenever possible and reserve blue-green for rare
   changes. This reduces cost but cannot provide reliable rollback for mixed
   mappings or incompatible field changes.
2. Let deployment scripts own migration state. This is easy to start but loses
   durable coordination, fencing, and operator visibility after interruptions.
3. Stop old-target writes immediately after cutover. This reduces load but makes
   rollback return to stale data and requires reconstructing the missed interval.

## Consequences

- Migrations can pause, resume, abort, and roll back without guessing state.
- Dual-write load and storage persist through the rollback window.
- MongoDB metadata needs retention, access control, fencing, and audit support.
- Cutover is slower because verification and source/cache catch-up are explicit.
- Retirement is a separate authorized operation rather than an automatic side
  effect of alias movement.

## Validation

- A failed pre-cutover migration leaves the active index unaffected.
- A post-cutover rollback returns to a current, queryable old target.
- Lost Redis routing state is reconstructed from MongoDB metadata.
- Expired leases cannot continue migration transitions.
- Verification gates block alias swaps when mappings, high-watermarks, cache
  generations, target transformation identity, deletes, or sampled canonical
  data are incorrect.
- Alias swaps and operator actions are atomic where supported and fully audited.
- No target is retired while referenced, needed for rollback, or missing required
  retention and rebuild evidence.
- Concurrent migrations for the same projection are fenced safely.

## Review trigger

Revisit if dual-write cost violates capacity objectives, rollback detection takes
longer than the retention window, or a sink/index change cannot support equivalent
fencing and verification.

## Related concepts

- [Bootstrap consistency](bootstrap-consistency.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Redis authority boundary](../01-system-context/redis-authority-boundary.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Reconciliation and repair](reconciliation-and-repair.md)
- [Disaster recovery](disaster-recovery.md)

## Follow-up questions

- What rollback-window duration matches detection and remediation objectives?
- Which roles may approve cutover, rollback, forced abort, and retirement?
- What exact retention and backup evidence is required before retirement?
