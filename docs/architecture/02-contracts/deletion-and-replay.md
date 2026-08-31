---
type: Architecture Review Topic
title: Deletion and replay
description: Defines root deletion, nested removal, tombstones, and late-event behavior.
tags: [contracts, deletion, replay, tombstones]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Deletion and replay

## Decision to stamp

Define how every relationship shape reacts to deletes and how long stale events
are prevented from resurrecting removed state.

## Draft proposal

Represent deletes as first-class versioned events. Root deletes remove or mark
the projection; child deletes remove only the matching nested member; reference
deletes follow an explicit null/default/error policy. Persist deletion fences for
at least the maximum replay and Kafka retention horizon, not a fixed five-minute
cache TTL.

## Pros

- Makes replay convergence symmetric for creates, updates, and deletes.
- Prevents delayed events from silently resurrecting data.
- Forces downstream deletion and privacy requirements into the design.

## Cons and risks

- Durable tombstones can grow without bound.
- Hard deletion reduces diagnostic evidence and repair options.
- Retention-safe cleanup needs watermarks or source compaction guarantees.

## Questions to stamp

- Are soft deletes part of source schemas or engine policy?
- What proves a deletion fence is safe to discard?
- How are projection documents rebuilt when a deleted child is absent from a
  source snapshot?
