---
type: Glossary
title: Domain language
description: Establishes consistent terms for the architecture review.
tags: [context, glossary]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Domain language

## Decision to stamp

Agree on terms that otherwise blur business data, event transport, and derived
storage.

## Draft proposal

- **Source entity:** MongoDB-owned business record.
- **CDC event:** Kafka envelope describing a source-entity change.
- **Projection:** Denormalized document shape defined by a manifest.
- **Root entity:** Entity whose identity becomes the Elasticsearch document ID.
- **Related entity:** Child, reference, or multi-hop entity contributing fields.
- **Physical index:** Versioned Elasticsearch index receiving writes.
- **Search alias:** Stable read name pointing to one accepted physical index.
- **Active write targets:** Physical indices receiving stream writes.
- **Bootstrap:** Full projection construction from source-of-truth state.
- **Reconciliation:** Comparison and optional repair of projected state.

## Pros

- Reduces accidental conflation of persistence and source-of-truth roles.
- Gives later contracts stable nouns.

## Cons and risks

- Terms may conflict with existing organization vocabulary.
- "Child" can imply ownership that does not exist across services.

## Questions to stamp

- Which terms already have canonical meanings in the target organization?
- Should migration and repair use separate terms for control-plane operations?
