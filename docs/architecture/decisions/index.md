# Decision register

Accepted decisions:

- [ADR-0001: Live ingestion source](0001-live-ingestion-source.md) - Kafka is
  the engine's sole live CDC input.
- [ADR-0002: Redis authority boundary](0002-redis-authority-boundary.md) - Redis
  is non-authoritative for projector progress and control-plane state.
- [ADR-0003: Durable state ownership](0003-durable-state-ownership.md) - Kafka
  owns cursors and engine-owned MongoDB owns control-plane and DLQ state.
- [ADR-0004: Debezium envelope boundary](0004-debezium-envelope-boundary.md) -
  Existing Debezium records are consumed as-is and unprocessable records go to
  the durable DLQ.
- [ADR-0005: Identity and ordering scope](0005-identity-and-ordering-scope.md) -
  Kafka tracks delivery while source-scoped metadata fences entity freshness.
- [ADR-0006: Relationship expansion scope](0006-relationship-expansion-scope.md) -
  V1 uses rooted acyclic expansion graphs while allowing scalar cross-references.
- [ADR-0007: Transformation execution boundary](0007-transformation-execution-boundary.md) -
  Bloblang-in-Go is authoritative and Painless is limited to fenced mutations.
- [ADR-0008: Deletion and replay semantics](0008-deletion-and-replay-semantics.md) -
  Deletes are first-class fenced events with durable root and child deletion
  custody.
- [ADR-0009: Projection schema ownership](0009-projection-schema-ownership.md) -
  Manifests own explicit versioned mappings while MongoDB remains authoritative
  for engine metadata.
- [ADR-0010: Elasticsearch write semantics](0010-elasticsearch-write-semantics.md) -
  Bulk mutations are versioned, source-fenced, idempotent, and explicitly
  complete across pinned dual-write targets.
- [ADR-0011: Kafka offset and delivery semantics](0011-kafka-offset-and-delivery-semantics.md) -
  Commits use at-least-once, highest-contiguous-completion semantics with
  durable terminal dispositions.
- [ADR-0012: Backpressure, retry, and DLQ policy](0012-backpressure-retry-dlq-policy.md) -
  Bounded queues, mode-specific retries, scoped blocking, and protected DLQ
  custody contain failures without data loss.
- [ADR-0013: Stream pipeline context resolution](0013-stream-pipeline-context-resolution.md) -
  Bounded pending, controlled source fallback, and deterministic coalescing
  resolve live events without indefinite waits.
- [ADR-0014: Cache and reverse-lookup semantics](0014-cache-and-reverse-lookup-semantics.md) -
  Redis uses relation-specific derived caches, controlled read-through, and
  generation-based rebuilds.
- [ADR-0015: Bootstrap consistency and handoff](0015-bootstrap-consistency-and-handoff.md) -
  Zero-downtime bootstrap uses source cluster-time boundaries, live dual-write,
  overlap replay, and resumable checkpoints.
- [ADR-0016: Blue-green migration lifecycle](0016-blue-green-migration-lifecycle.md) -
  Migrations use durable fenced state, live dual-write, verified alias cutover,
  rollback windows, and approved retirement.
- [ADR-0017: Schema evolution compatibility](0017-schema-evolution-compatibility.md) -
  Projection changes use conservative semantic classification, with only
  allow-listed additive mappings applied in place and meaning-changing changes
  migrated blue-green.
- [ADR-0018: Reconciliation and repair](0018-reconciliation-and-repair.md) -
  Reconciliation compares MongoDB-derived canonical projections with pinned
  Elasticsearch targets and repairs only through the normal fenced write path.
- [ADR-0019: Disaster recovery](0019-disaster-recovery.md) - Recovery uses
  authority-aware failure procedures, backs up engine metadata, and rebuilds
  Redis and Elasticsearch from MongoDB plus retained CDC.
- [ADR-0020: Performance and capacity](0020-performance-and-capacity.md) -
  Capacity uses workload-specific objectives, safety budgets, effective mutation
  accounting, and benchmark validation.
- [ADR-0021: Availability and scaling](0021-availability-and-scaling.md) -
  Search, ingestion, and recovery availability are separated; workload-group
  consumers degrade by scope and scale on sustained multi-signal pressure.
- [ADR-0022: Security and privacy](0022-security-and-privacy.md) - The engine
  uses explicit field classifications, least privilege, environment-delivered
  secrets, protected diagnostics, source-driven anonymization, and
  retention-aware derived data.
- [ADR-0023: Compatibility and dependencies](0023-compatibility-and-dependencies.md) -
  Supported dependency combinations are capability-tested, pinned per release,
  and fail readiness when required behavior is unavailable.
- [ADR-0024: Telemetry conventions](0024-telemetry-conventions.md) -
  OpenTelemetry uses stable resource identity, bounded dimensions, protected
  diagnostics, sampled traces, and non-blocking collector export.
- [ADR-0025: Metrics and alerting](0025-metrics-and-alerting.md) - Canonical
  metrics cover progress, saturation, dependencies, correctness, and lifecycle;
  symptom-based alerts use bounded dimensions and owned runbooks.
- [ADR-0026: Tracing and logging](0026-tracing-and-logging.md) - Tracing follows
  meaningful processing boundaries, ordinary event spans are sampled, and
  structured logs use protected diagnostics without raw payloads.
- [ADR-0027: Health and diagnostics](0027-health-and-diagnostics.md) - Liveness
  is local-only, readiness is stable and scoped, and authenticated diagnostics
  expose authoritative projection state with timestamps.
- [ADR-0028: Modes and configuration](0028-modes-and-configuration.md) -
  Mode-specific executables share a release line, initially ship in one image,
  and run as explicit Kubernetes workloads with validated immutable
  configuration.
- [ADR-0029: Deployment and orchestration](0029-deployment-and-orchestration.md) -
  Kubernetes schedules lifecycle-specific workloads while MongoDB durably
  coordinates migrations, checkpoints, leases, and fencing.
- [ADR-0030: Runbooks and intervention](0030-runbooks-and-intervention.md) -
  Production operation requires indexed, exercised runbooks with explicit
  automation boundaries and audited intervention.
- [ADR-0031: Ownership and governance](0031-ownership-and-governance.md) -
  A small-team model assigns back-end, lead, DevOps, and DPO responsibilities
  with risk-based review and explicit escalation paths.
- [ADR-0032: Correctness invariants](0032-correctness-invariants.md) -
  Correctness uses strict safety invariants, conditional convergence claims,
  explicit unknown outcomes, and narrowest-safe-scope failure handling.
- [ADR-0033: Test strategy](0033-test-strategy.md) - Layered per-change,
  real-dependency integration, and release testing proves accepted contracts;
  scheduled resilience testing is deferred.
- [ADR-0034: Performance and failure testing](0034-performance-and-failure-testing.md) -
  Representative workload benchmarks and targeted failure tests validate
  capacity and recovery; recurring resilience campaigns remain deferred.
- [ADR-0035: Release acceptance](0035-release-acceptance.md) - Separate gates
  promote binaries, manifests, and migrations using correctness, compatibility,
  performance, security, and rollback evidence, with an audited emergency path.
- [ADR-0036: Data-store responsibilities](0036-data-store-responsibilities.md) -
  Domain and engine MongoDB own authoritative state while Kafka, Redis, and
  Elasticsearch have explicit delivery or derived roles.
- [ADR-0037: Objectives and boundaries](0037-objectives-and-boundaries.md) -
  The engine owns Kafka-driven projection processing and lifecycle coordination;
  source capture, business workflows, query APIs, and cluster administration
  remain external.
- [ADR-0038: System guarantees](0038-system-guarantees.md) - The engine
  promises bounded at-least-once delivery, conditional convergence, measurable
  freshness, verified migration continuity, and explicit scoped exceptions.
- [ADR-0039: Domain language](0039-domain-language.md) - A canonical glossary
  distinguishes business records, CDC transport, logical projections, physical
  targets, and engine metadata.
- [ADR-0040: Manifest contract](0040-manifest-contract.md) - Versioned manifests
  declare projection semantics and are validated structurally and semantically
  before source progress, including multi-projection event fan-out.
- [ADR-0041: Runtime topology](0041-runtime-topology.md) - Stream workload
  groups use compatible capacity and failure domains with bounded
  per-projection work and partition-contiguous completion.
- [ADR-0042: Startup and readiness](0042-startup-and-readiness.md) - Deterministic
  mode-specific preflight prevents unsafe source progress, while scoped
  degradation and stable probes avoid unnecessary restart and rebalance storms.
- [ADR-0043: Source-draft conflict resolution](0043-source-draft-conflict-resolution.md) -
  Runtime foreign-database reads are denied by default and allowed only through
  explicit, bounded relation-level fallback policies.
- [ADR-0044: Architecture review process](0044-architecture-review-process.md) -
  A single maintainer uses decision cards, a mandatory follow-up register, and
  milestone design gates without requiring a second approver.
- [ADR-0045: Fallback-read observability and privacy](0045-fallback-read-observability.md) -
  Fallback reads use bounded metrics and trace correlation; raw identifiers stay
  out of ordinary telemetry, with controlled references reserved for
  authenticated diagnostics.
- [ADR-0046: Source fallback eligibility](0046-source-fallback-eligibility.md) -
  V1 allows only explicitly bounded, unique-index direct reference fallback;
  reverse-index misses use rebuild, repair, or pending, and fallback stays off
  until numeric source budgets are evidenced.
- [ADR-0047: Debezium operation and snapshot scope](0047-debezium-operation-and-snapshot-scope.md) -
  V1 applies only live create/update/delete events, durably DLQs snapshot-read
  and unknown operations, and uses the engine bootstrap path for existing data.
- [ADR-0048: Event size and buffer diagnostics](0048-event-size-and-buffer-diagnostics.md) -
  Deterministic per-event size/decode failures use durable DLQ custody and
  bounded diagnostics; temporary queue saturation pauses intake.
- [ADR-0049: Transaction metadata treatment](0049-transaction-metadata-treatment.md) -
  Transaction metadata is optional diagnostic context in v1, not a freshness
  fence or cross-document atomicity contract.
- [ADR-0050: Deletion-fence retention policy](0050-deletion-fence-retention-policy.md) -
  Durable deletion fences remain for at least 30 days; older raw replay is
  unsupported and carries an explicitly accepted resurrection risk after expiry.
- [ADR-0051: Rebuilding child membership from source snapshots](0051-child-rebuild-from-source-snapshots.md) -
  Child absence is authoritative only after complete source-bounded enumeration;
  ambiguous completeness or ordering blocks rebuild cutover.
- [ADR-0052: MongoDB source-ordering scope](0052-mongodb-source-ordering-scope.md) -
  Freshness compares `source.ts_ms` + `source.ord` within one source, replica
  set, and entity scope; production field availability is an enablement gate.
- [ADR-0053: Canonical source ID serialization](0053-canonical-source-id-serialization.md) -
  Entity IDs use type-tagged, length-framed payloads with explicit ObjectID,
  UUID, string, and signed BSON integer representations.
- [ADR-0054: Equal source revision conflicts](0054-equal-source-revision-conflicts.md) -
  Identical redelivery at one revision is a no-op; differing content is retained
  for entity-local authoritative repair without an invented tie-breaker.
- [ADR-0055: Derived Elasticsearch fence metadata](0055-derived-elasticsearch-fence-metadata.md) -
  Elasticsearch stores rebuildable, private per-entity fence entries in a
  reserved `engine_meta` namespace while MongoDB remains authoritative.
- [ADR-0056: Nested relation capacity boundaries](0056-nested-relation-capacity-boundaries.md) -
  Authors own model and expected size; alerts cover estimate overruns and
  benchmarked hard limits block unsafe work without dropping valid data.
- [ADR-0057: Reference change propagation](0057-reference-change-propagation.md) -
  Mutable reference changes use reverse-indexed durable bounded recomputation
  of affected roots with contributor-scoped fencing.
- [ADR-0058: Reference fan-out execution threshold](0058-reference-fanout-execution-threshold.md) -
  Per-relation benchmark-derived ceilings route small fan-outs to live updates
  and larger valid fan-outs to durable deferred recomputation.
- [ADR-0059: Relation lifecycle classification](0059-relation-lifecycle-classification.md) -
  Every relation declares snapshot, owned-child, or independent-entity
  semantics separately from its Elasticsearch storage shape.
- [ADR-0060: Bloblang subset and resource budgets](0060-bloblang-subset-and-resource-budgets.md) -
  Transformations use a deterministic data-only allowlist; numeric resource
  caps require representative benchmark evidence.
- [ADR-0061: Projection dependency invalidation](0061-projection-dependency-invalidation.md) -
  V1 recomputes at entity/relation granularity through the declared dependency
  graph, even when a changed field is not used by the projection.
- [ADR-0062: Transformation version and target association](0062-transformation-version-target-association.md) -
  Each physical target pins its manifest hash, transformation version, and
  normalized transformation/allowlist digest; semantic changes use a new
  blue-green target.
- [ADR-0063: Independent-reference delete effects](0063-independent-reference-delete-effects.md) -
  Recompute dependent roots on reference deletion, remove optional copied
  fields or apply explicit safe missing-value behavior, and never cascade-delete
  independent roots.
- [ADR-0064: Fence token encoding and protection](0064-fence-token-encoding-and-protection.md) -
  Entity keys and mutation fingerprints use domain-separated HMAC-SHA-256
  tokens with target-pinned key IDs, fixed encoding, and pseudonymous-data
  handling.

- [ADR template](template.md) - Copy only after a concept has been reviewed and
  stamped.
