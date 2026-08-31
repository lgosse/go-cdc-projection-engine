---
type: Architecture Decision Record
title: "ADR-0031: Ownership and governance"
description: A small-team model assigns back-end, lead, DevOps, and DPO responsibilities with risk-based review.
tags: [architecture, adr, operations, ownership, governance, privacy]
status: accepted
decision_id: ADR-0031
accepted_on: 2026-08-31
owner: TBD
conditions:
  - The back-end team owns source-contract assumptions, manifests, projections, and normal engine operation.
  - The back-end team and lead software engineer own engine architecture, runtime semantics, compatibility, and high-risk lifecycle decisions.
  - DevOps owns Kubernetes, deployment pipelines, data-service infrastructure, telemetry plumbing, and infrastructure access.
  - The Data Protection Officer owns privacy, retention, data-protection, and legal interpretations.
  - Review is risk-based; mandatory two-person approval is not required for every operation.
  - Role references and escalation paths are recorded even when one person temporarily fills multiple roles.
---

# ADR-0031: Ownership and governance

## Context

The organization has a small back-end team, one lead software engineer, and a
small DevOps function. It does not have separate engine, source-domain,
projection, platform, or security-governance teams. Ownership must therefore be
explicit without creating artificial review layers or assuming unavailable
capacity. Privacy and legal interpretation belongs to the Data Protection
Officer (DPO), while technical controls remain implemented by back-end and
DevOps roles.

## Decision

The back-end team owns source-contract assumptions, manifest semantics,
relationships, transformations, projection correctness, and normal engine
operation. The back-end team and lead software engineer own runtime semantics,
compatibility, architecture, and the shared engine.

The lead software engineer is accountable for high-risk architecture,
compatibility, migration, rollback, and retirement decisions and may request
DevOps help for infrastructure concerns. DevOps owns Kubernetes, deployment
pipelines, Kafka/MongoDB/Redis/Elasticsearch infrastructure, telemetry plumbing,
capacity primitives, and infrastructure access. The DevOps senior manager is an
escalation and organizational support role, not an assumed day-to-day
implementer.

The DPO owns privacy, data-protection, retention, and legal interpretations.
Back-end and DevOps roles implement the resulting technical controls.

Apply risk-based review:

1. Routine changes receive back-end peer review and automated validation.
2. Cross-cutting or infrastructure-impacting changes receive back-end review
   plus lead approval, with DevOps consultation when infrastructure is affected.
3. High-risk or privacy-sensitive changes receive lead approval, explicit
   operational evidence, and DPO involvement when privacy, retention, erasure,
   or protected fields are affected.

This does not require two-person approval for every operation. Privileged actions
remain authenticated, attributable, and audited.

Each projection or deployment records role references such as:

```text
technical_owner: backend
architecture_escalation: lead
infrastructure_escalation: devops
privacy_owner: dpo
```

An unowned projection or missing escalation path is not deployable. The back-end
team owns engine and projection correctness and freshness objectives. DevOps owns
infrastructure availability and capacity primitives. The lead resolves
cross-cutting trade-offs. Cost is reviewed jointly using effective mutations,
dual-write load, source reads, and cache traffic.

The back-end team proposes deprecation and verifies consumers and replacements;
the lead approves high-risk retirement; DevOps executes infrastructure cleanup;
and the DPO is consulted when retained data, erasure, or retention obligations
are affected.

## Alternatives considered

1. **Create separate specialized teams.** This would provide nominal boundaries
   but does not match the organization.
2. **Require lead approval for every change.** This centralizes accountability
   but slows routine delivery and encourages bypasses.
3. **Give back-end ownership of infrastructure.** This reduces coordination but
   overloads the team and weakens platform expertise and access controls.
4. **Leave ownership implicit.** This appears lightweight but creates ambiguous
   escalation and abandoned projections.

## Consequences

- Manifests and deployments need role references and escalation paths.
- Review requirements depend on change risk and affected concerns.
- Back-end and DevOps need cross-training and runbooks for concentrated duties.
- DPO consultation is part of privacy, retention, erasure, and protected-field
  changes.
- Stronger role separation can be introduced later without changing the basic
  ownership interfaces.

## Validation

- Every production projection identifies technical, architecture,
  infrastructure, and privacy ownership or escalation paths.
- Routine changes do not wait for unnecessary approval.
- Infrastructure-impacting changes receive DevOps review.
- High-risk and privacy-sensitive changes have required lead/DPO evidence.
- Privileged actions are attributable and auditable.
- Deprecation and retirement identify who proposes, approves, and executes each
  stage.
- Missing or stale ownership blocks deployment or raises an actionable alert.

## Review triggers

Revisit if team size, on-call coverage, infrastructure ownership, privacy
requirements, or change volume makes role concentration unsafe or creates a
persistent review bottleneck.

## Related concepts

- [Ownership and governance](../07-operations/ownership-and-governance.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Runbooks and intervention](../07-operations/runbooks-and-intervention.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
