# Architecture Update Log

## 2026-08-31

- **Initialization**: Split the source drafts into an OKF v0.2 review bundle.
- **Creation**: Added provisional proposals, trade-offs, missing topics, and an
  empty decision register.
- **Decision**: Accepted ADR-0008 for durable, source-fenced root and child
  deletion semantics across replay, bootstrap, and reconciliation.
- **Decision**: Accepted ADR-0009 for explicit versioned projection mappings,
  strict production schemas, and Mongo-authoritative engine metadata.
- **Decision**: Accepted ADR-0010 for source-fenced idempotent bulk writes,
  explicit dual-write completion, and differentiated permanent versus
  infrastructure failures.
- **Decision**: Accepted ADR-0011 for at-least-once contiguous Kafka commits,
  durable terminal custody, audited operator skips, and bounded rebalance
  draining.
- **Decision**: Accepted ADR-0012 for bounded backpressure, mode-specific retry
  budgets, scoped failure blocking, and protected DLQ custody.
- **Decision**: Accepted ADR-0013 for bounded child-before-parent handling,
  controlled source-of-truth fallback, relation-specific cache misses, and
  deterministic coalescing.
- **Decision**: Accepted ADR-0014 for distinct reference and reverse-index
  policies, controlled read-through, and generation-based Redis rebuilds.
- **Decision**: Accepted ADR-0015 for zero-downtime bootstrap using source
  cluster-time boundaries, live dual-write, overlap replay, delete reconciliation,
  and resumable chunk checkpoints.
- **Decision**: Accepted ADR-0016 for durable fenced migration state, live
  dual-write rollback windows, explicit verification gates, and approved
  retirement.
- **Decision**: Accepted ADR-0017 for conservative semantic schema evolution,
  allow-listed additive in-place updates, and blue-green migration for
  meaning-changing or ambiguous changes.
- **Decision**: Accepted ADR-0018 for pinned-target canonical reconciliation,
  layered drift detection, and opt-in repairs through the normal fenced write
  path.
- **Decision**: Accepted ADR-0019 for authority-aware disaster recovery,
  metadata backup/PITR, derived-state rebuilds, and fail-closed handling of
  missing history or deletion evidence.
- **Decision**: Accepted ADR-0020 for workload-specific performance objectives,
  hard safety budgets, effective mutation accounting, and benchmark-gated
  capacity planning.
- **Decision**: Accepted ADR-0021 for separate search/ingestion/recovery health,
  workload-group consumer isolation, scoped degradation, and multi-signal
  scaling.
- **Decision**: Accepted ADR-0022 with conditions for explicit field
  classifications, least privilege, environment-delivered secrets, source-
  driven anonymization, TTL-based derived-data expiry, and audited single-
  operator actions; live key rotation and fence-identifier treatment remain
  future topics.
- **Decision**: Accepted ADR-0023 for a capability-tested dependency matrix,
  pinned client/runtime versions, fail-closed readiness checks, and guarded
  rolling upgrades.
- **Decision**: Accepted ADR-0024 for bounded OpenTelemetry attributes,
  protected diagnostics, sampled traces, shared error codes, and non-blocking
  collector export.
- **Decision**: Accepted ADR-0025 for canonical progress/saturation/correctness
  metrics and symptom-based alerts with bounded dimensions, ownership, and
  runbooks.
- **Decision**: Accepted ADR-0026 for meaningful trace boundaries, sampled event
  spans, retained failure/control-plane traces, protected structured logs, and
  non-blocking telemetry export.
- **Decision**: Accepted ADR-0027 for local liveness, stable scoped readiness,
  separate search/ingestion/recovery status, and authenticated authoritative
  diagnostics.
