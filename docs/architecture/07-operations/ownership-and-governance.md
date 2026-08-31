---
type: Architecture Review Topic
title: Ownership and governance
description: Assigns responsibility for contracts, manifests, infrastructure, and lifecycle decisions.
tags: [operations, ownership, governance]
status: proposed
---

# Ownership and governance

## Decision to stamp

Assign owners and approval boundaries across the shared engine and domain-specific
projections.

## Draft proposal

The engine team owns runtime semantics, manifest schema, compatibility, and core
operations. Source-domain teams own CDC completeness, source fields, relation
meaning, and projection correctness. The search platform owns Elasticsearch
capacity and access; the data/platform team owns Kafka, Redis, telemetry, and
deployment primitives. Manifest changes require both domain and engine review
when they affect cost, fan-out, mappings, or transformations.

## Pros

- Avoids a generic platform silently owning business semantics.
- Makes cross-team incident routing explicit.
- Shared review catches both domain and runtime consequences.

## Cons and risks

- Multi-owner changes can slow delivery.
- Organizational boundaries may not match the proposed split.
- Unowned projections can become stale operational liabilities.

## Questions to stamp

- Which actual teams fill each role?
- Who owns SLOs and production cost?
- What deprecation process removes unused projections and manifests?
