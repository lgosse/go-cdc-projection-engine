---
type: Architecture Review Topic
title: Transformation contract
description: Defines supported derivations and consistent execution across stream and batch modes.
tags: [contracts, transformations, bloblang, correctness]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Transformation contract

## Decision to stamp

Choose one semantic model for transformations and define when dependent values
must be recomputed.

## Draft proposal

Compile a restricted Bloblang subset at startup and execute it in Go against a
canonical assembled projection in every mode. Elasticsearch scripts should only
apply idempotent, fenced mutations; they should not implement a second
transformation language.

## Pros

- Stream, bootstrap, audit, and repair share one transformation implementation.
- Avoids a difficult and potentially incomplete Bloblang-to-Painless compiler.
- Startup compilation catches syntax errors before consumption.

## Cons and risks

- A partial child event may not contain enough context to re-evaluate a whole
  projection.
- Complex scripts can undermine manifest safety and performance.
- Canonical assembly in stream mode may require more cached state than drafted.

## Questions to stamp

- What exact Bloblang operations and resource limits are allowed?
- Which dependency changes trigger recomputation?
- How are transformation versions associated with projected documents?
