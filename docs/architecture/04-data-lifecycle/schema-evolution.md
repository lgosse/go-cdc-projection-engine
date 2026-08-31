---
type: Architecture Review Topic
title: Schema evolution
description: Defines compatibility classes for manifests and Elasticsearch mappings.
tags: [lifecycle, schema, compatibility, elasticsearch]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Schema evolution

## Decision to stamp

Define semantic compatibility and which changes may happen in place versus
requiring a new physical index.

## Draft proposal

Classify changes as runtime-only, backward-compatible additive, reindex-required,
or forbidden. Compare normalized Elasticsearch mappings and settings
semantically. Allow only an explicit safe subset of additive mapping changes to
apply automatically; require a blue-green migration for analyzers, incompatible
types, removals, or changed transformation meaning.

## Pros

- Avoids fragile byte-level mapping comparisons.
- Makes automatic mutation conservative and reviewable.
- Couples transformation semantics to schema versioning.

## Cons and risks

- Compatibility depends on consumer query behavior, not only ES rules.
- Some settings are dynamic while others require index recreation.
- Conservative classification causes more migrations.

## Questions to stamp

- Who approves automatic mapping updates?
- How are old and new manifests coordinated during rollout?
- What query-contract compatibility must be preserved?
