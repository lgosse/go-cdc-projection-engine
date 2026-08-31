# Operations

Stamp how humans and automation configure, deploy, and own the engine.

- [Modes and configuration](modes-and-configuration.md) - Operational commands,
  configuration precedence, and safety; separate mode-specific executables,
  explicit Kubernetes workloads, and validated immutable configuration are
  accepted in [ADR-0028](../decisions/0028-modes-and-configuration.md).
- [Deployment and orchestration](deployment-and-orchestration.md) - Kubernetes
  workload shapes and lifecycle coordination; lifecycle-specific workloads,
  durable MongoDB migration coordination, and graceful stream rollouts are
  accepted in [ADR-0029](../decisions/0029-deployment-and-orchestration.md).
- [Runbooks and intervention](runbooks-and-intervention.md) - Required incident
  and maintenance procedures; indexed, exercised runbooks and explicit
  intervention boundaries are accepted in
  [ADR-0030](../decisions/0030-runbooks-and-intervention.md).
- [Ownership and governance](ownership-and-governance.md) - Responsibility for
  manifests, source contracts, platform dependencies, and decisions; the
  small-team back-end, lead, DevOps, and DPO ownership model is accepted in
  [ADR-0031](../decisions/0031-ownership-and-governance.md).

The remaining operational follow-ups are concrete team and rotation
assignments, deployment-tool selection, Kubernetes policy details, runbook
cadence, cost budgets, and deprecation evidence. Resolve them in the linked
concepts without reopening the accepted boundaries unless a review trigger is
met.
