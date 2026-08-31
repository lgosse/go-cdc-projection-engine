# Data lifecycle

Stamp how a projection is created, evolved, verified, repaired, and retired.

- [Bootstrap consistency](bootstrap-consistency.md) - Snapshot boundaries,
  partitioning, and convergence with live traffic; zero-downtime cluster-time
  handoff is accepted for v1.
- [Schema evolution](schema-evolution.md) - Compatibility classification and
  automatic changes.
- [Blue-green migration](blue-green-migration.md) - Durable migration state,
  dual writes, verification, and cutover.
- [Reconciliation and repair](reconciliation-and-repair.md) - Drift detection,
  comparison, and remediation.
- [Disaster recovery](disaster-recovery.md) - Loss scenarios and restoration
  procedures.
