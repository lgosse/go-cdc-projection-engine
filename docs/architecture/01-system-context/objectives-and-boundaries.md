---
type: Architecture Review Topic
title: Objectives and boundaries
description: Defines what the projection engine owns and deliberately does not own.
tags: [context, scope, boundaries]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Objectives and boundaries

## Decision to stamp

Define the engine's users, owned outcomes, external dependencies, and non-goals.

## Draft proposal

Build a stateless, configuration-driven Go service that consumes CDC envelopes
from Kafka, assembles denormalized projections using MongoDB-backed domain data
and Redis lookups, and maintains Elasticsearch read models. Domain services own
source data and CDC publication; query clients own use of the published search
aliases.

Keep CDC capture, source writes, business workflows, public query APIs, and
Elasticsearch cluster administration outside the engine.

## Pros

- Establishes a reusable engine rather than a domain-specific indexer.
- Keeps source ownership with existing services.
- Makes projections disposable and rebuildable.

## Cons and risks

- A generic engine can push domain complexity into manifests and scripts.
- End-to-end correctness depends on upstream CDC contracts outside its control.
- "Stateless" is misleading unless checkpoints and migration state have clear
  durable owners.

## Questions to stamp

- Who authors and approves manifests?
- Are query APIs and index templates strictly out of scope?
- What scale and number of projections define the first useful release?
