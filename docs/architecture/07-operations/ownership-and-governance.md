---
type: Architecture Review Topic
title: Ownership and governance
description: Assigns responsibility for contracts, manifests, infrastructure, and lifecycle decisions.
tags: [operations, ownership, governance]
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

# Ownership and governance

## Decision

Use a small-team ownership model rather than inventing separate specialized
teams. The back-end team owns source-contract assumptions, manifest semantics,
relationships, transformations, projection correctness, and normal engine
operation. The back-end team and lead software engineer own runtime semantics,
compatibility, architecture, and the shared engine.

The lead software engineer is accountable for high-risk architecture,
compatibility, migration, rollback, and retirement decisions. The lead may
request help from DevOps when infrastructure expertise is needed. DevOps owns
Kubernetes, deployment pipelines, Kafka/MongoDB/Redis/Elasticsearch
infrastructure, telemetry plumbing, capacity primitives, and infrastructure
access. The DevOps senior manager is an escalation and organizational support
role, not an assumed day-to-day implementer.

The Data Protection Officer (DPO) owns privacy, data-protection, retention, and
legal interpretations. Back-end and DevOps roles implement the resulting
technical controls.

Apply three review levels:

1. **Routine:** back-end peer review and automated validation for bounded,
   non-breaking changes.
2. **Cross-cutting or infrastructure-impacting:** back-end review plus lead
   approval, with DevOps consultation when deployment, capacity, networking, or
   dependency behavior is affected.
3. **High-risk or privacy-sensitive:** lead approval, explicit operational
   evidence, and DPO involvement when privacy, retention, erasure, or protected
   fields are affected.

This is risk-based governance, not mandatory two-person approval for every
operation. Privileged actions remain authenticated, attributable, and audited.

Record role references for each projection or deployment, for example:

```text
technical_owner: backend
architecture_escalation: lead
infrastructure_escalation: devops
privacy_owner: dpo
```

Role references remain valid when one person temporarily fills multiple roles.
An unowned projection or missing escalation path is not deployable.

The back-end team owns engine and projection correctness and freshness
objectives. DevOps owns infrastructure availability and capacity primitives. The
lead resolves trade-offs that span both. Cost is reviewed jointly, using engine
measurements such as effective mutations, dual-write load, source reads, and
cache traffic.

The back-end team proposes projection deprecation and verifies consumers and
replacement behavior. The lead approves high-risk retirement. DevOps executes
infrastructure cleanup. The DPO is consulted when retained data, erasure, or
retention obligations are affected.

## Pros

- Matches the actual organization instead of creating artificial team boundaries.
- Keeps business and projection semantics with the back-end team.
- Gives the lead a clear escalation role without requiring approval for every
  routine change.
- Assigns infrastructure and privacy accountability explicitly.
- Preserves a path to stronger separation if the organization grows.

## Cons and risks

- The back-end team carries a broad operational scope.
- Lead review can become a bottleneck for high-risk changes.
- DevOps capacity and availability are concentrated in a small team.
- One person may fill multiple roles, weakening separation of duties.
- Concrete ownership assignments still need to be maintained as people change.

## Alternatives considered

1. Create separate engine, source-domain, projection, and platform teams. This
   would provide clearer nominal boundaries but does not match the organization.
2. Make the lead approve every change. This centralizes accountability but slows
   routine delivery and encourages approval bypasses.
3. Let the back-end team own infrastructure as well. This reduces coordination
   but overloads the team and weakens platform expertise and access controls.
4. Leave ownership implicit. This appears lightweight but creates ambiguous
   escalation and abandoned projections.

## Consequences

- Manifests and deployments need role references and escalation paths.
- Review and approval requirements depend on change risk and affected concerns.
- Back-end and DevOps must maintain cross-training and runbooks for concentrated
  responsibilities.
- DPO consultation is part of privacy, retention, erasure, and protected-field
  changes.
- Governance can evolve toward stronger role separation without changing the
  ownership model's basic interfaces.

## Validation

- Every production projection identifies technical, architecture,
  infrastructure, and privacy ownership or an explicit escalation path.
- Routine changes do not wait for unnecessary cross-team approval.
- Infrastructure-impacting changes receive DevOps review.
- High-risk and privacy-sensitive changes have the required lead/DPO evidence.
- Privileged actions are attributable and auditable.
- Deprecation and retirement steps identify who proposes, approves, and executes
  each stage.
- A missing or stale owner blocks deployment or raises an actionable alert.

## Review trigger

Revisit if team size, on-call coverage, infrastructure ownership, privacy
requirements, or change volume makes the current role concentration unsafe or
creates a persistent review bottleneck.

## Related concepts

- [Modes and configuration](modes-and-configuration.md)
- [Deployment and orchestration](deployment-and-orchestration.md)
- [Runbooks and intervention](runbooks-and-intervention.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
