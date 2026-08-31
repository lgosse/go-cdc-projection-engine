---
type: Architecture Review Process
title: Architecture review and stamping method
description: Defines how provisional architecture concepts become accepted decisions.
tags: [architecture, review, governance]
sources:
  - resource: ../design/system.md
    title: System design draft
  - resource: ../design/otel.md
    title: OpenTelemetry design draft
status: proposed
---

# Architecture review and stamping method

## Decision to stamp

Agree on how we review, record, and revisit architecture decisions before any
implementation structure is created.

## Draft proposal

Review one concept at a time. Replace its provisional text with the agreed
decision, alternatives considered, consequences, verification criteria, owner,
and review date. Then create an ADR in the [decision register](decisions/) and
change the concept status to `accepted`.

Statuses are `proposed`, `in-review`, `accepted`, `superseded`, and `rejected`.

## Pros

- Keeps decisions small, discoverable, and independently editable.
- Makes uncertainty and unresolved dependencies visible.
- Separates source-draft proposals from reviewed architecture.

## Cons and risks

- Requires disciplined cross-link and status maintenance.
- Concepts reviewed in isolation can hide system-level trade-offs.

## Stamp checklist

- State the decision and non-goals precisely.
- Record at least one viable alternative and why it was not selected.
- Identify correctness invariants and operational failure modes.
- Define measurable acceptance evidence.
- Check linked concepts for consequences or contradictions.
- Record an owner and a future review trigger.
