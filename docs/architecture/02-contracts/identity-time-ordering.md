---
type: Architecture Review Topic
title: Identity, time, and ordering
description: Defines canonical keys and deterministic stale-event comparison.
tags: [contracts, identity, ordering, correctness]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Identity, time, and ordering

## Decision to stamp

Choose canonical ID encoding and an ordering token that can safely fence stale
updates.

## Draft proposal

Define typed canonical string encodings for ObjectID, UUID, strings, and numeric
IDs. Use a monotonic source revision or CDC log position as the primary fence;
use source timestamps only when their clock and granularity guarantees are
explicit. Store ordering metadata per contributing entity, not only at the root.

## Pros

- Avoids lossy timestamp-only last-write-wins behavior.
- Makes replay and comparison deterministic across modes.
- Prevents key collisions across heterogeneous sources.

## Cons and risks

- A universally comparable source revision may not exist across collections.
- Per-entity metadata increases projection size and script complexity.
- Type-prefixed IDs may differ from current consumer expectations.

## Questions to stamp

- Which upstream ordering tokens are available today?
- Are equal revisions possible and how are ties resolved?
- Is ordering needed across entities or only within one entity's history?
