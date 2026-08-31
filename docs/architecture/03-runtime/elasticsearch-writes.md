---
type: Architecture Review Topic
title: Elasticsearch writes
description: Defines atomic update, fencing, bulk, and dual-write behavior.
tags: [runtime, elasticsearch, bulk, idempotency]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Elasticsearch writes

## Decision to stamp

Define the smallest atomic write, idempotency mechanism, partial-result handling,
and how active write targets are selected.

## Draft proposal

Use bulk scripted upserts keyed by projection document ID. Scripts apply already
transformed mutations and compare per-entity fences before changing fields. Pin
the active target set for each batch, inspect every bulk item result, and require
success or an explicitly terminal disposition for every target before advancing
source progress.

## Pros

- Handles duplicate and stale deliveries without read-before-write races.
- Item-level inspection contains mapping and document-specific failures.
- A pinned target set avoids intra-batch migration ambiguity.

## Cons and risks

- Generated scripts and nested-array scans may be expensive and hard to audit.
- Dual-write success can diverge across indices.
- `retry_on_conflict` does not replace correct idempotency or fencing.

## Questions to stamp

- Are stored scripts, generated scripts, or whole-document replacements allowed?
- What atomic rule applies when one dual-write target succeeds and another fails?
- How are script versions rolled out with manifests?
