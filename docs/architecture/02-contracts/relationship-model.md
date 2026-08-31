---
type: Architecture Review Topic
title: Relationship model
description: Defines supported joins and dependency propagation across source entities.
tags: [contracts, relationships, joins, fanout]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Relationship model

## Decision to stamp

Define supported relationship shapes, ownership, cardinality, and how a related
entity change finds every affected root projection.

## Draft proposal

Initially support root fields, one-to-one cached references, one-to-many nested
children, and explicitly declared multi-hop relations. Require every non-root
source to declare a durable or rebuildable root-resolution strategy and bounded
fan-out behavior. Reject ambiguous cycles and unbounded many-to-many graphs.

## Pros

- Covers the examples without pretending to be a general graph engine.
- Makes reverse-index requirements explicit.
- Validation can reject relationships the runtime cannot update safely.

## Cons and risks

- Reference updates such as organization changes may fan out to many roots.
- Cached reverse maps can be large and expensive to rebuild.
- Limiting graph shapes may exclude important projections.

## Questions to stamp

- How are reference-field changes propagated to existing root documents?
- What fan-out limit triggers deferred rebuild instead of live updates?
- Are relations snapshots, owned children, or independent entities?
