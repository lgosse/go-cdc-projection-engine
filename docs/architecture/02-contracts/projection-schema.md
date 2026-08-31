---
type: Architecture Review Topic
title: Projection schema
description: Defines ownership and limits of the Elasticsearch document shape.
tags: [contracts, elasticsearch, schema, mappings]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Projection schema

## Decision to stamp

Define how logical projection fields map to Elasticsearch documents, including
metadata, nested limits, and consumer compatibility.

## Draft proposal

The manifest owns explicit mappings and index settings for a versioned physical
index. Reserve an engine metadata namespace for manifest version, source fences,
and reconciliation data. Require bounded nested cardinalities and prohibit
dynamic mapping for production projections unless explicitly accepted.

## Pros

- Prevents accidental field-type drift.
- Engine metadata supports deterministic writes and audits.
- Bounded nested data makes capacity risks reviewable.

## Cons and risks

- Strict mappings increase the operational cost of new fields.
- Hidden metadata increases document and update size.
- Elasticsearch nested documents can be expensive even below hard limits.

## Questions to stamp

- Which fields are public contract versus internal metadata?
- Are consumers compatible with versioned mapping changes?
- When should large child sets become separate indices instead of nested arrays?
