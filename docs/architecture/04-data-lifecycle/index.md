# Data lifecycle

Stamp how a projection is created, evolved, verified, repaired, and retired.

- [Bootstrap consistency](bootstrap-consistency.md) - Snapshot boundaries,
  partitioning, and convergence with live traffic; zero-downtime cluster-time
  handoff is accepted for v1.
- [Schema evolution](schema-evolution.md) - Compatibility classification and
  automatic changes; conservative semantic classification is accepted in
  ADR-0017.
- [Blue-green migration](blue-green-migration.md) - Durable migration state,
  dual writes, verification, and cutover; fenced rollback and approved
  retirement are accepted for v1.
- [Reconciliation and repair](reconciliation-and-repair.md) - Drift detection,
  comparison, and remediation; pinned canonical audits and fenced opt-in repair
  are accepted in ADR-0018.
- [Disaster recovery](disaster-recovery.md) - Loss scenarios and restoration
  procedures; authority-aware recovery and derived-state rebuilds are accepted
  in ADR-0019.
