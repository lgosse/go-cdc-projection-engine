---
type: Architecture Review Topic
title: Blue-green migration
description: Defines a recoverable state machine for provisioning, dual-write, validation, and cutover.
tags: [lifecycle, migration, dual-write, elasticsearch]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: in-review
---

# Blue-green migration

## Decision to stamp

Define the durable migration state machine, fencing of concurrent operators, and
safe rollback at every phase.

## Draft proposal

Model migration as durable, idempotent phases: planned, target provisioned,
dual-write acknowledged, bootstrap running, verification passed, alias cut over,
old target quiesced, and retirement approved. Store state outside ephemeral Redis
or make it reconstructible from a signed desired-state artifact plus Elasticsearch.
Never delete the old index automatically at cutover.

## Pros

- Supports retry and operator inspection after partial failures.
- Keeps rollback available through a retention window.
- Explicit worker acknowledgment closes the race before bootstrap.

## Cons and risks

- Adds a control-plane component or stricter deployment orchestration.
- Long dual-write windows increase load and divergence surface.
- Rollback after new-only writes may require reverse migration.

## Questions to stamp

- Which system owns and fences migration leadership?
- What verification is required before alias swap?
- Who approves retirement and after what retention period?
