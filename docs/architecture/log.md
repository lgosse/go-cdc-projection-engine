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

## 2026-09-24

- **Decision**: Accepted ADR-0045 for bounded fallback-read metrics and trace
  correlation, excluding raw identifiers from ordinary telemetry and reserving
  controlled pseudonymous references for authenticated diagnostics when needed.
- **Decision**: Accepted ADR-0046 for v1 unique-index direct-reference fallback,
  fail-closed limits, and rebuild/repair/pending behavior for reverse-index
  misses; exact numeric budgets remain blocking for the Contract package.
- **Decision**: Accepted ADR-0047 for v1 Debezium event handling: apply only
  `c`/`u`/`d`, durably DLQ `r` and unknown operations, and populate existing
  data through the engine bootstrap path. Q-010 is deferred until snapshot-read
  support is proposed.
- **Decision**: Accepted ADR-0048 for oversized and unbufferable event
  diagnostics: deterministic per-event size/decode failures reach durable DLQ
  custody before offset completion; temporary queue saturation pauses intake.
  Metrics remain bounded, and detailed source context stays in protected
  diagnostics.
- **Decision**: Accepted ADR-0049 for v1 transaction metadata: absent, null, or
  object values are handled as optional context; transaction fields do not
  define freshness, cause the engine to wait for related events, or promise
  cross-document atomicity.
- **Decision**: Accepted ADR-0050 for deletion-fence retention: retain durable
  fences for at least 30 days, disallow raw replay beyond that window, and
  rebuild from current source before writes resume after an out-of-window
  recovery. The residual risk of resurrection after fence expiry is accepted;
  DLQ and backup retention details remain separate follow-ups.
- **Decision**: Accepted ADR-0051 for rebuilding nested child membership:
  source absence is authoritative only after complete enumeration at a recorded
  boundary; preserve known child fences and block cutover when scan completeness
  or stale-event ordering is uncertain.
- **Decision**: Accepted ADR-0052 for MongoDB source ordering: require
  `source.ts_ms` + `source.ord`, scoped by source name, replica set, database,
  collection, and entity; defer missing or incomparable revisions. Verify the
  deployed production plugin and event fields before enabling a source (Q-123).
- **Decision**: Accepted ADR-0053 for canonical source IDs: use type-tagged,
  length-framed payloads; preserve exact string bytes; distinguish signed BSON
  integer widths; and fail closed for unsupported or mismatched types. UUID
  legacy subtype 3 requires an explicit byte-order configuration.
- **Decision**: Accepted ADR-0054 for equal source revisions: identical source
  mutations are duplicate no-ops; differing mutations at the same scoped
  revision are durably retained and sent to entity-local authoritative repair,
  without a Kafka, arrival-time, processing-time, or payload-order tie-break.
  Production source behavior remains evidence-gated by Q-123.

## 2026-09-25

- **Decision**: Accepted ADR-0055 for derived Elasticsearch fence metadata:
  reserve root `engine_meta` with per-contributor scoped entity key, source
  revision, deletion marker, and mutation fingerprint. MongoDB remains
  authoritative, and public responses omit the rebuildable Elasticsearch copy.
  Key/fingerprint encoding and privacy protection remain open under Q-124.
- **Decision**: Accepted ADR-0056 for nested relation capacity: projection
  authors choose the storage model and declare expected volume; expected-bound
  overruns alert, while benchmarked hard caps block unsafe work without DLQing
  valid data. Numeric limits remain evidence-gated by Q-070.
- **Decision**: Accepted ADR-0057 for reference-change propagation: use the
  reverse index to durably schedule bounded recomputation of affected roots,
  fence relation-derived writes by contributor revision, and repair an
  incomplete index before publishing from it; ADR-0058 defines its fan-out
  execution threshold.
- **Decision**: Accepted ADR-0058 for reference fan-out: each relation uses a
  benchmark-derived live-update ceiling; larger fan-outs become durable deferred
  recomputation work rather than rejected or DLQed source data. Numeric values
  remain a production capacity evidence gate under Q-070.
- **Decision**: Accepted ADR-0059 for relation lifecycle: each non-root relation
  declares snapshot, owned-child, or independent-entity semantics separately
  from storage shape. Owned-child events cannot resurrect roots protected by a
  deletion fence; independent-entity deletes do not cascade, while copied-field
  behavior remains open under Q-126.
- **Decision**: Accepted ADR-0060 for Bloblang transformations: use a deterministic,
  versioned data-only allowlist over canonical projection context; validate
  mapping complexity before source progress, and set numeric event, document,
  collection, execution, and worker-resource caps from representative profiles
  under Q-069/Q-070 before production.
- **Decision**: Accepted ADR-0061 for dependency invalidation: every accepted
  source update to a participating entity recomputes its dependent roots through
  the declared graph, even if the changed field is unused. Owned-child changes
  affect their owner; independent-reference updates use reverse-indexed durable
  work. Cache-only and unrelated-source changes do not trigger recomputation.
- **Decision**: Accepted ADR-0062 for transformation version association: each
  physical target pins its immutable manifest hash, transformation version, and
  normalized transformation/allowlist digest. Semantic changes use a new
  blue-green target, with target-specific transforms during dual-write; no
  public per-document version marker is required. Broader version-family
  separation remains open under Q-057.
- **Decision**: Accepted ADR-0063 for independent-reference deletion: use the
  reverse index to recompute dependent roots without cascading; remove copied
  optional fields or apply explicitly safe missing-value behavior. Block
  manifests with required reference-derived outputs and no safe missing rule;
  large fan-out remains durable deferred work under ADR-0058.
- **Decision**: Accepted ADR-0064 for derived Elasticsearch fence tokens: use
  domain-separated HMAC-SHA-256 over canonical scoped identities and logical
  source mutations, encode fixed-size results as unpadded Base64URL, and pin a
  nonsecret key ID and format version per physical target. Keep keys available
  through blue-green rollback; treat tokens as pseudonymous, with legal
  retention still deferred under Q-076.
- **Follow-up re-scoped**: Defer Q-122's numeric source-fallback budgets until
  before the first fallback-enabled relation. Keep fallback off meanwhile;
  during Step 3a, select a representative lookup and prepare source-capacity
  measurements. Record evidence-backed per-relation and aggregate limits
  before enabling fallback.
