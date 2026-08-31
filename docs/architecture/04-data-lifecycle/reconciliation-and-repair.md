---
type: Architecture Review Topic
title: Reconciliation and repair
description: Defines authoritative comparison, sampling, drift classification, and repair safety.
tags: [lifecycle, audit, drift, repair]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Reconciliation and repair

## Decision to stamp

Define what is compared, how samples are selected, and when automated repair is
safe.

## Draft proposal

Assemble canonical documents from MongoDB source state using the same manifest
and transformation engine, normalize non-semantic fields, and compare against a
pinned physical index. Combine risk-based and random sampling, classify drift by
field and likely cause, and make auto-repair opt-in with the same fencing rules as
normal writes.

## Pros

- Detects silent data loss and transformation divergence.
- Field-level classification is more actionable than hash mismatch alone.
- Pinned targets avoid alias changes during an audit.

## Cons and risks

- Cross-database reads may observe inconsistent moments.
- Sampling cannot prove the absence of drift.
- Automated repair can mask systemic faults or overwrite newer data.

## Questions to stamp

- What sampling strategy meets confidence and cost goals?
- Which drift classes may auto-repair?
- How are deletes and expected transient differences normalized?
