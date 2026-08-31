---
type: Architecture Review Topic
title: Bootstrap consistency
description: Defines how a full source scan converges with concurrent CDC traffic.
tags: [lifecycle, bootstrap, mongodb, consistency]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Bootstrap consistency

## Decision to stamp

Define the snapshot boundary, scan partitioning, and proof that concurrent live
changes and deletes are neither lost nor overwritten.

## Draft proposal

Record an explicit CDC boundary, build the target from a consistent-enough
MongoDB snapshot, and replay or dual-write all later changes until caught up.
Partition scans using stable keyset boundaries discovered from sampled split
points rather than arithmetic min/max assumptions. Apply the same source fences
as stream writes.

## Pros

- Makes bootstrap correctness independent of job timing.
- Keyset ranges work with ObjectIDs and uneven key distributions.
- Shared fencing prevents old snapshot data overwriting newer changes.

## Cons and risks

- Cross-database snapshots cannot be globally transactional.
- Capturing and replaying a boundary requires CDC connector support.
- Deleted records absent from the snapshot still require explicit handling.

## Questions to stamp

- What consistency can each MongoDB source actually provide?
- Does bootstrap consume Kafka, rely on dual-write, or do both?
- How are resumable checkpoints represented for failed chunks?
