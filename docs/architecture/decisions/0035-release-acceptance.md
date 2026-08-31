---
type: Architecture Decision Record
title: "ADR-0035: Release acceptance"
description: Separate gates promote engine binaries, manifests, and migrations using correctness, compatibility, performance, security, and rollback evidence.
tags: [architecture, adr, validation, release, acceptance]
status: accepted
decision_id: ADR-0035
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Engine binaries, manifest changes, and projection migrations use separate acceptance gates.
  - Mandatory evidence includes correctness, compatibility, affected performance, security/privacy review, and rollback behavior at the appropriate risk level.
  - Migration cutover and retirement require explicit authorization and configured observation/rollback evidence.
  - Emergency changes use a documented audited break-glass path and never bypass absolute correctness invariants or deletion fences.
  - Exact subtype thresholds, observation durations, and emergency response deadlines remain operational follow-ups.
---

# ADR-0035: Release acceptance

## Context

The engine releases binaries, manifests, and physical projection targets with
different risk profiles. Accepted decisions already define layered testing,
representative performance evidence, dependency compatibility, conservative
schema evolution, blue-green migration, small-team governance, and explicit
operator runbooks. A single undifferentiated release checklist would either
over-test routine changes or under-test migrations and emergency actions.

## Decision

Use separate acceptance gates.

Engine binary releases require formatting and static analysis, unit/property and
affected integration tests, compatibility-matrix validation against required and
production dependencies, no regression against accepted freshness/recovery or
hard safety limits, security and dependency review, rolling-upgrade evidence,
and updated runbooks.

Manifest and projection changes require schema and semantic validation,
deterministic compatibility classification, focused contract/transformation/
query tests, performance evidence when fan-out, nested cardinality, document
size, or source-read cost changes, DPO review for privacy/retention/erasure or
protected fields, and migration evidence when the change is reindex-required.
Persisted semantic or operational-cost changes are not configuration-only.

Projection migrations require a provisioned and validated target, live dual-write
before bootstrap, boundary-overlap and catch-up evidence,
document/field/delete/fence/mapping validation, query smoke tests,
reconciliation evidence, rollback evidence, a configured observation/rollback
window without unresolved critical correctness issues, and explicit lead
accountability before cutover or retirement. DevOps executes infrastructure
actions while MongoDB remains authoritative for migration evidence and state.

Provide a documented break-glass path for service restoration, security
incidents, or data-protection incidents. Emergency changes are limited to the
smallest safe scope, never bypass absolute correctness invariants or deletion
fences, use an authorized operator, record reason/actor/scope/artifact/outcome,
receive retrospective tests and review, and involve the DPO when privacy or
retention is affected.

Routine changes receive back-end peer review. Infrastructure-impacting changes
receive lead approval and DevOps consultation. Migration cutover, rollback,
forced abort, and retirement are explicit authorized operations with lead
accountability; privacy-sensitive changes involve the DPO. Two-person approval
is not mandatory.

Do not hard-code a universal observation duration yet. A configured rollback
window must demonstrate stable freshness and lag, no unresolved critical alerts,
successful target and delete/fence checks, reconciliation evidence, and
old-target protection until approval.

## Alternatives considered

1. **One universal release checklist.** This either over-tests routine changes or
   under-tests migrations.
2. **Approve only code changes.** This misses risky manifest, mapping,
   transformation, and cost changes.
3. **Let emergency changes bypass all gates.** This can corrupt data or violate
   privacy obligations.
4. **Require a fixed long observation period for every migration.** This delays
   safe changes and ignores measured detection and recovery behavior.

## Consequences

- Each release artifact records change class, evidence, approvals, and
  manifest/projection versions.
- Compatibility, correctness, performance, and security checks become promotion
  inputs rather than post-deployment checks.
- Migration cutover and retirement require explicit evidence and authorization.
- Emergency changes need a break-glass runbook and retrospective review.
- Exact subtype thresholds, observation duration, and emergency deadlines remain
  operational follow-ups.

## Validation

- A change matrix maps each subtype to mandatory tests, approvals, and artifacts.
- Routine releases cannot bypass compatibility and correctness gates.
- Manifest changes with persisted semantic impact enter migration acceptance.
- Cutover and retirement require documented evidence and authorization.
- Emergency changes remain auditable and preserve absolute safety invariants.
- Release evidence links the deployed artifact, dependency versions, and manifest
  versions.

## Review triggers

Revisit if release lead time becomes excessive, incidents reveal a missing gate,
emergency changes become routine, rollback evidence is insufficient, or ownership
and privacy requirements change.

## Related concepts

- [Release acceptance](../08-validation/release-acceptance.md)
- [Test strategy](../08-validation/test-strategy.md)
- [Performance and failure testing](../08-validation/performance-and-failure-testing.md)
- [Correctness invariants](../08-validation/correctness-invariants.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Ownership and governance](../07-operations/ownership-and-governance.md)
- [Runbooks and intervention](../07-operations/runbooks-and-intervention.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
