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
- **Decision**: Accepted ADR-0028 for separate mode-specific executables built
  from shared libraries, initially packaged in one image, and run as explicit
  Kubernetes workloads with validated immutable configuration.
- **Decision**: Accepted ADR-0029 for lifecycle-specific Kubernetes workloads,
  durable MongoDB migration coordination, graceful stream rollouts, and an
  initially deferred permanent migration controller.
- **Decision**: Accepted ADR-0030 for indexed and exercised runbooks, bounded
  automatic recovery, explicit operator intervention, role-based ownership, and
  auditable operational evidence.
- **Decision**: Accepted ADR-0031 for a small-team ownership model: back-end
  ownership, lead accountability, DevOps infrastructure responsibility, DPO
  privacy authority, and risk-based review.
- **Decision**: Accepted ADR-0032 for strict safety invariants, conditional
  convergence claims, explicit `unknown` comparison outcomes, and
  narrowest-safe-scope handling; global-stop thresholds for shared metadata
  failures remain deferred.
- **Decision**: Accepted ADR-0033 for layered per-change, integration, and
  release testing against accepted invariants and real dependency capabilities;
  scheduled resilience testing remains a later topic.
- **Decision**: Accepted ADR-0034 for representative workload benchmarks,
  targeted failure tests, isolated destructive testing, and initial release
  gates; recurring resilience campaigns remain deferred.
- **Decision**: Accepted ADR-0035 for separate binary, manifest, and migration
  acceptance gates, explicit cutover/retirement evidence, and an audited
  emergency path that preserves absolute safety invariants.
- **Decision**: Accepted ADR-0036 for a consolidated datastore authority matrix,
  MongoDB canonicality, Kafka cursor ownership, rebuildable Redis/Elasticsearch,
  and explicit required reverse-index treatment.
- **Decision**: Accepted ADR-0037 for the engine ownership boundary, explicit
  non-goals, allow-listed source fallback, and an evidence-gated first release
  with one representative projection.
- **Decision**: Accepted ADR-0038 for bounded at-least-once delivery,
  conditional convergence, measurable freshness and recovery objectives,
  verified migration continuity, and explicit scoped exceptions.
- **Decision**: Accepted ADR-0039 for canonical terminology distinguishing
  source records, CDC transport, logical projections, physical targets, and
  engine metadata, with related entity as the neutral relation term.
- **Decision**: Accepted ADR-0040 for versioned semantic manifests, layered
  preflight validation, strict deployment/semantic ownership, and independent
  outcomes for multi-projection event fan-out.
- **Decision**: Accepted ADR-0041 for workload-class stream groups, bounded
  per-partition and per-projection work, multi-projection dispatch, and
  partition-contiguous completion with selective consumer-group isolation.
- **Decision**: Accepted ADR-0042 for deterministic mode-specific startup
  preflight, scoped projection blocking, stable readiness during recoverable
  dependency outages, and bounded graceful termination.
- **Decision**: Accepted ADR-0043 to reconcile the source drafts: foreign
  database reads are denied by default and permitted only through explicit,
  bounded relation-level cache-miss fallback.
- **Decision**: Accepted ADR-0044 for a single-maintainer review process with
  explicit decision cards, a durable follow-up register, milestone gates, and
  design checkpoints before substantial implementation.
