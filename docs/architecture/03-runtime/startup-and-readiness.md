---
type: Architecture Review Topic
title: Startup and readiness
description: Defines safe startup validation and degraded-operation behavior.
tags: [runtime, startup, readiness, schema]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Startup and readiness

## Decision to stamp

Define the ordered startup handshake, which failures are fatal, and whether one
blocked projection may coexist with healthy ones.

## Draft proposal

Load and schema-validate all configuration, compile transformations, validate
relationship graphs, verify dependency capabilities, and compare desired versus
live Elasticsearch state before consuming. Expose process liveness separately
from workload readiness. Permit projection-level blocking only if partition and
health reporting remain unambiguous.

## Pros

- Prevents consumption under a known-invalid configuration.
- Separates a live process from one able to make projection progress.
- Projection isolation can preserve healthy workloads.

## Cons and risks

- Live mapping comparison is more complex than textual equality.
- Pausing shared topics can accidentally block unrelated projections.
- Continuous polling can hide a condition that needs operator action.

## Questions to stamp

- Is readiness all-or-nothing or reported per projection?
- Which mapping changes may be applied automatically?
- What timeout changes dependency slowness into startup failure?
