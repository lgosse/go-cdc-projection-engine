---
type: Architecture Review Topic
title: Manifest contract
description: Defines the projection manifest's scope, versioning, and validation boundary.
tags: [contracts, manifest, configuration]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Manifest contract

## Decision to stamp

Define what a manifest may express, how it is versioned, and what must fail at
validation rather than at runtime.

## Draft proposal

Use one versioned YAML manifest per logical projection, validated against a
versioned JSON Schema plus semantic graph checks. It declares source topics,
Mongo collections for offline modes, identity and timestamp fields,
relationships, target mappings, and bounded transformations. Secrets remain
environment references rather than literal values.

## Pros

- Keeps the engine domain-agnostic and reviewable.
- Enables validation before consumers start.
- A schema version allows controlled evolution of engine capabilities.

## Cons and risks

- A single large manifest can become a domain-specific programming language.
- JSON Schema alone cannot validate relationship cycles or semantic compatibility.
- Embedding full Elasticsearch settings tightly couples manifests to one sink.

## Questions to stamp

- What belongs in manifests versus deployment configuration?
- Can one source event update multiple projections?
- What compatibility policy applies to manifest-schema versions?
