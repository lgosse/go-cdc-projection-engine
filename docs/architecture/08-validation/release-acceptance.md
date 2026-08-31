---
type: Architecture Review Topic
title: Release acceptance
description: Defines evidence and gates for promoting engine and manifest changes.
tags: [validation, release, acceptance]
status: proposed
---

# Release acceptance

## Decision to stamp

Define separate promotion gates for engine binaries, manifest changes, and
projection migrations.

## Draft proposal

Require formatting/static analysis, unit/property tests, contract validation,
supported-version integration tests, upgrade/downgrade compatibility evidence,
capacity-regression checks, security review for new data access, and a tested
rollback plan. Migration cutover additionally requires source-boundary catch-up,
document/field validation, query smoke tests, and an observation window.

## Pros

- Prevents count-only migration verification.
- Recognizes that manifests can be as risky as code.
- Makes rollback evidence part of acceptance rather than incident improvisation.

## Cons and risks

- Comprehensive gates increase lead time and infrastructure cost.
- Query correctness may require consumer-owned test suites.
- Strict gates need an explicit emergency-change process.

## Questions to stamp

- Which evidence is mandatory for each change class?
- Who approves migration cutover and emergency exceptions?
- What observation window is required before old-index retirement?
