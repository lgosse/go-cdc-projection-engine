---
type: Architecture Review Topic
title: Correctness invariants
description: Defines the properties that implementations must preserve across duplicates, disorder, and failure.
tags: [validation, correctness, invariants]
status: proposed
---

# Correctness invariants

## Decision to stamp

State the architecture's core properties independently of implementation.

## Draft proposal

At minimum: duplicate delivery is idempotent; an older entity revision cannot
overwrite a newer one; deletion cannot be undone by an older event; committed
offsets have a durable terminal outcome for every required target; bootstrap plus
post-boundary changes converges to source truth; migration cutover exposes one
verified read target; and identical canonical source state yields identical
projection content.

## Pros

- Gives every later implementation choice a shared oracle.
- Enables property and model-based tests for disorder and crashes.
- Separates correctness from throughput optimizations.

## Cons and risks

- Some invariants require stronger upstream metadata than currently specified.
- Cross-database source snapshots make "source state" time-dependent.
- Proving convergence for transformations and fan-out may be expensive.

## Questions to stamp

- Which invariants are absolute versus bounded by retention or availability?
- What canonical state is used when sources cannot share a snapshot?
- Which invariant violations stop processing immediately?
