---
type: Architecture Follow-up Register
title: Architecture implementation follow-ups
description: Tracks unresolved questions from accepted decisions through implementation milestones.
tags: [architecture, follow-ups, implementation, governance]
status: active
owner: Project maintainer
---

# Architecture implementation follow-ups

This register prevents accepted decisions from becoming a false signal that
every implementation detail is already known. It is the durable queue for
questions discovered during design and coding.

## Operating rules

- Add a row as soon as a new question is discovered; do not keep it only in
  personal notes or an issue description.
- Classify each item as `blocking-v1`, `blocking-milestone`, or `deferred`.
- A blocking item must be resolved before its dependent milestone is marked
  complete. Resolution means an ADR/concept update, an explicit narrowed scope,
  or evidence that removes the question.
- A deferred item requires a safe interim behavior, one owner, and a concrete
  trigger or milestone for resumption.
- Link the resulting ADR, concept section, test, or operational evidence in the
  final column.
- At each milestone start and end, review this table and record the outcome in
  the implementation plan or commit notes.

## Exact question coverage

Audit performed 2026-09-01: 122 question occurrences were found in architecture question sections, representing 121 unique questions. Every occurrence is represented below; identical wording from multiple sections is combined in one row with all section labels shown. Questions under both “Follow-up questions” and “Questions to stamp” are included so no unresolved prompt is lost.

| ID | Source section | Exact question | Class | Gate | Owner | Status | Theme |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Q-001 | [cdc-event-envelope.md](02-contracts/cdc-event-envelope.md) (Questions to stamp) | Which Debezium operation codes and snapshot modes are supported in the first release? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-002 | [cdc-event-envelope.md](02-contracts/cdc-event-envelope.md) (Questions to stamp) | What is the fallback priority among Kafka key, `after._id`, and `before._id` when resolving entity identity? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-003 | [cdc-event-envelope.md](02-contracts/cdc-event-envelope.md) (Questions to stamp) | What metric, log event, and diagnostic fields identify a record that exceeds a later size limit or cannot be safely buffered? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-004 | [cdc-event-envelope.md](02-contracts/cdc-event-envelope.md) (Questions to stamp) | How should transaction metadata be treated when present? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-005 | [deletion-and-replay.md](02-contracts/deletion-and-replay.md) (Questions to stamp) | What proves a deletion fence is safe to discard? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-006 | [deletion-and-replay.md](02-contracts/deletion-and-replay.md) (Questions to stamp) | How are projection documents rebuilt when a deleted child is absent from a source snapshot? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-007 | [identity-time-ordering.md](02-contracts/identity-time-ordering.md) (Questions to stamp) | Which exact Debezium source fields are available in every production connector configuration, and what is their comparison scope? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-008 | [identity-time-ordering.md](02-contracts/identity-time-ordering.md) (Questions to stamp) | What canonical serialization is required for ObjectID, UUID, string, and numeric IDs? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-009 | [identity-time-ordering.md](02-contracts/identity-time-ordering.md) (Questions to stamp) | Are equal source revisions possible, and what deterministic tie-breaker is available? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-010 | [identity-time-ordering.md](02-contracts/identity-time-ordering.md) (Questions to stamp) | Should snapshot-read events participate in normal fencing or use a separate bootstrap path? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-011 | [projection-schema.md](02-contracts/projection-schema.md) (Follow-up questions) | What exact metadata namespace and derived-copy fields are required for local fencing? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-012 | [projection-schema.md](02-contracts/projection-schema.md) (Follow-up questions) | When should large child sets become separate indices instead of nested arrays? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-013 | [relationship-model.md](02-contracts/relationship-model.md) (Questions to stamp) | How are reference-field changes propagated to existing root documents? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-014 | [relationship-model.md](02-contracts/relationship-model.md) (Questions to stamp) | What fan-out limit triggers deferred rebuild instead of live updates? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-015 | [relationship-model.md](02-contracts/relationship-model.md) (Questions to stamp) | Are relations snapshots, owned children, or independent entities? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-016 | [transformation-contract.md](02-contracts/transformation-contract.md) (Questions to stamp) | What exact Bloblang operations and resource limits are allowed? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-017 | [transformation-contract.md](02-contracts/transformation-contract.md) (Questions to stamp) | Which dependency changes trigger recomputation? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-018 | [transformation-contract.md](02-contracts/transformation-contract.md) (Questions to stamp) | How are transformation versions associated with projected documents? | blocking-v1 | Contract package | Project maintainer | open | Contract package |
| Q-019 | [backpressure-retry-dlq.md](03-runtime/backpressure-retry-dlq.md) (Follow-up questions) | What numeric queue, age, and elapsed-time budgets meet each mode's objective? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-020 | [backpressure-retry-dlq.md](03-runtime/backpressure-retry-dlq.md) (Follow-up questions) | What exact DLQ indexes, retention schedule, and redaction rules are required? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-021 | [backpressure-retry-dlq.md](03-runtime/backpressure-retry-dlq.md) (Follow-up questions) | Which dependency signals determine projection-, partition-, or workload-level blocking? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-022 | [cache-and-reverse-lookups.md](03-runtime/cache-and-reverse-lookups.md) (Follow-up questions) | Which keys may expire and which must be retained? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-023 | [cache-and-reverse-lookups.md](03-runtime/cache-and-reverse-lookups.md) (Follow-up questions) | What snapshot boundary and generation marker are required for a rebuild? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-024 | [cache-and-reverse-lookups.md](03-runtime/cache-and-reverse-lookups.md) (Follow-up questions) | What staleness is acceptable for each reference type? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-025 | [cache-and-reverse-lookups.md](03-runtime/cache-and-reverse-lookups.md) (Follow-up questions) | Which relation types may use controlled source-of-truth read-through? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-026 | [elasticsearch-writes.md](03-runtime/elasticsearch-writes.md) (Follow-up questions) | What exact tombstone representation and retention cleanup policy are required? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-027 | [elasticsearch-writes.md](03-runtime/elasticsearch-writes.md) (Follow-up questions) | Which operator authorization permits a terminal disposition for a failed target? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-028 | [offsets-and-delivery.md](03-runtime/offsets-and-delivery.md) (Follow-up questions) | What default drain deadline and retry budgets meet the recovery objective? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-029 | [offsets-and-delivery.md](03-runtime/offsets-and-delivery.md) (Follow-up questions) | What exact unique-key and retention policy should the DLQ metadata use? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-030 | [startup-and-readiness.md](03-runtime/startup-and-readiness.md) (Follow-up questions) | Which checks are mandatory for each mode (`stream`, `bootstrap`, `repair`, `replay`, and `migration`)? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-031 | [startup-and-readiness.md](03-runtime/startup-and-readiness.md) (Follow-up questions) | If a running worker loses engine-owned MongoDB connectivity, should it remain ready-but-paused for a bounded grace period, or become unready immediately because safe participation cannot be confirmed? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-032 | [startup-and-readiness.md](03-runtime/startup-and-readiness.md) (Follow-up questions) | What exact rule determines whether a failure blocks one projection, a Kafka partition, a workload group, or the whole deployment? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-033 | [startup-and-readiness.md](03-runtime/startup-and-readiness.md) (Follow-up questions) | What revalidation cadence and bounded termination deadline should be used? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-034 | [stream-pipeline.md](03-runtime/stream-pipeline.md) (Follow-up questions) | What pending age/retry limit and durable custody trigger apply before an event becomes an orphan or repair case? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-035 | [stream-pipeline.md](03-runtime/stream-pipeline.md) (Follow-up questions) | Which relation types permit controlled source-of-truth read-through on a cache miss, and how are those limits represented in the manifest? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-036 | [stream-pipeline.md](03-runtime/stream-pipeline.md) (Follow-up questions) | What are the maximum batch age, size, memory, and hot-key fairness budgets? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-037 | [stream-pipeline.md](03-runtime/stream-pipeline.md) (Follow-up questions) | Can coalescing cross topic or partition boundaries safely when one combined mutation fails? | blocking-milestone | Runtime design | Project maintainer | open | Runtime design |
| Q-038 | [blue-green-migration.md](04-data-lifecycle/blue-green-migration.md) (Follow-up questions) | What rollback-window duration matches detection and remediation objectives? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-039 | [blue-green-migration.md](04-data-lifecycle/blue-green-migration.md) (Follow-up questions) | Which roles may approve cutover, rollback, forced abort, and retirement? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-040 | [blue-green-migration.md](04-data-lifecycle/blue-green-migration.md) (Follow-up questions) | What exact retention and backup evidence is required before retirement? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-041 | [bootstrap-consistency.md](04-data-lifecycle/bootstrap-consistency.md) (Follow-up questions) | How is each connector's `source.wallTime` watermark acquired and paired with replay positions in production? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-042 | [bootstrap-consistency.md](04-data-lifecycle/bootstrap-consistency.md) (Follow-up questions) | What exact chunk checkpoint schema and operator resume controls are required? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-043 | [bootstrap-consistency.md](04-data-lifecycle/bootstrap-consistency.md) (Follow-up questions) | What reconciliation evidence is sufficient to declare delete safety at cutover? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-044 | [disaster-recovery.md](04-data-lifecycle/disaster-recovery.md) (Follow-up questions) | What are recovery time and recovery point objectives per mode? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-045 | [disaster-recovery.md](04-data-lifecycle/disaster-recovery.md) (Follow-up questions) | How long must Kafka retention exceed the maximum outage and recovery window? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-046 | [disaster-recovery.md](04-data-lifecycle/disaster-recovery.md) (Follow-up questions) | What backup/PITR evidence makes metadata recovery valid? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-047 | [disaster-recovery.md](04-data-lifecycle/disaster-recovery.md) (Follow-up questions) | Who authorizes recovery when deletion fences or DLQ history cannot be restored? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-048 | [disaster-recovery.md](04-data-lifecycle/disaster-recovery.md) (Follow-up questions) | What regional failover procedure prevents two projector deployments from writing concurrently? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-049 | [reconciliation-and-repair.md](04-data-lifecycle/reconciliation-and-repair.md) (Follow-up questions) | What sampling strategy meets confidence and cost goals? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-050 | [reconciliation-and-repair.md](04-data-lifecycle/reconciliation-and-repair.md) (Follow-up questions) | What delay and evidence classify a difference as transient? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-051 | [reconciliation-and-repair.md](04-data-lifecycle/reconciliation-and-repair.md) (Follow-up questions) | Which drift classes may auto-repair, and who may enable it? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-052 | [reconciliation-and-repair.md](04-data-lifecycle/reconciliation-and-repair.md) (Follow-up questions) | What source-boundary evidence confirms deletes? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-053 | [reconciliation-and-repair.md](04-data-lifecycle/reconciliation-and-repair.md) (Follow-up questions) | What retention and reporting format are required for audit and repair history? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-054 | [schema-evolution.md](04-data-lifecycle/schema-evolution.md) (Follow-up questions) | Which roles approve automatic additive updates versus migrations? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-055 | [schema-evolution.md](04-data-lifecycle/schema-evolution.md) (Follow-up questions) | Which exact mapping additions belong to the safe allow-list? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-056 | [schema-evolution.md](04-data-lifecycle/schema-evolution.md) (Follow-up questions) | Must consumer compatibility be declared per field? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-057 | [schema-evolution.md](04-data-lifecycle/schema-evolution.md) (Follow-up questions) | Are manifest-schema, projection-schema, and transformation versions separate? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-058 | [schema-evolution.md](04-data-lifecycle/schema-evolution.md) (Follow-up questions) | What deprecation period is required for renamed or removed fields? | blocking-milestone | Lifecycle design | Project maintainer | open | Lifecycle design |
| Q-059 | [availability-and-scaling.md](05-quality-attributes/availability-and-scaling.md) (Follow-up questions) | What exact availability percentages and outage intervention thresholds should complement the freshness and recovery targets? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-060 | [availability-and-scaling.md](05-quality-attributes/availability-and-scaling.md) (Follow-up questions) | What default cooldown and stabilization windows fit each workload group? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-061 | [availability-and-scaling.md](05-quality-attributes/availability-and-scaling.md) (Follow-up questions) | When does strict projection isolation justify a separate consumer group? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-062 | [availability-and-scaling.md](05-quality-attributes/availability-and-scaling.md) (Follow-up questions) | What disruption budget and zone-spreading rules are required for production? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-063 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | What versions exist in target environments? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-064 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | Which clients and transformation runtime are acceptable dependencies? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-065 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | What backward-compatibility promise does the engine itself offer? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-066 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | What minimum support window and security-patch policy apply to each dependency? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-067 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | Which dependency failures block one workload versus the whole engine? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-068 | [compatibility-and-dependencies.md](05-quality-attributes/compatibility-and-dependencies.md) (Follow-up questions) | Which binary/manifest combinations are safe for rolling deployment? | blocking-v1 | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-069 | [performance-and-capacity.md](05-quality-attributes/performance-and-capacity.md) (Follow-up questions) | What benchmark datasets and projected workload distributions should be kept current? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-070 | [performance-and-capacity.md](05-quality-attributes/performance-and-capacity.md) (Follow-up questions) | What exact hard limits and warning thresholds replace the initial targets? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-071 | [performance-and-capacity.md](05-quality-attributes/performance-and-capacity.md) (Follow-up questions) | When must a high-cardinality nested relation become a separate projection? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-072 | [performance-and-capacity.md](05-quality-attributes/performance-and-capacity.md) (Follow-up questions) | Do observed workloads justify changing the two-times peak planning margin? | blocking-milestone | Quality baseline | Project maintainer | open | Quality and compatibility |
| Q-073 | [security-and-privacy.md](05-quality-attributes/security-and-privacy.md) (Follow-up questions) | What exact fields, if any, receive a future Restricted-field exception? | deferred | Quality baseline | Project maintainer | deferred | Quality and compatibility |
| Q-074 | [security-and-privacy.md](05-quality-attributes/security-and-privacy.md) (Follow-up questions) | What calendar retention values satisfy operational and compliance requirements? | deferred | Quality baseline | Project maintainer | deferred | Quality and compatibility |
| Q-075 | [security-and-privacy.md](05-quality-attributes/security-and-privacy.md) (Follow-up questions) | How should required reverse indexes handle privacy-driven expiry? | deferred | Quality baseline | Project maintainer | deferred | Quality and compatibility |
| Q-076 | [security-and-privacy.md](05-quality-attributes/security-and-privacy.md) (Follow-up questions) | What is the legal treatment of identifiers retained in deletion fences? | deferred | Quality baseline | Project maintainer | deferred | Quality and compatibility |
| Q-077 | [security-and-privacy.md](05-quality-attributes/security-and-privacy.md) (Follow-up questions) | When should operator permissions evolve into separated roles? | deferred | Quality baseline | Project maintainer | deferred | Quality and compatibility |
| Q-078 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Follow-up questions; Questions to stamp) | Which conditions remove a pod from service versus pause one projection? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-079 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Follow-up questions) | Who may access diagnostics and how are accesses audited? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-080 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Follow-up questions) | What status schema and compatibility policy should the diagnostic endpoint use? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-081 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Follow-up questions) | How stale may an observation be before it becomes `unknown`? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-082 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Follow-up questions) | How should migration and blue-green phases affect readiness? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-083 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Questions to stamp) | Who can access diagnostics? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-084 | [health-and-diagnostics.md](06-observability/health-and-diagnostics.md) (Questions to stamp) | What data is authoritative when diagnostics and backend state disagree? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-085 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | Which exact dimensions are permitted for partition-level progress? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-086 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | What sustained windows and thresholds implement each alert severity? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-087 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | Which alerts page immediately versus create tickets? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-088 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | How are maintenance, bootstrap, migration, and repair windows suppressed? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-089 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | What alert ownership and runbook format will operations use? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-090 | [metrics-and-alerting.md](06-observability/metrics-and-alerting.md) (Follow-up questions) | What metric retention and backend-specific recording rules are required? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-091 | [telemetry-conventions.md](06-observability/telemetry-conventions.md) (Follow-up questions) | Which attributes are mandatory at resource, process, workload, and event scope? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-092 | [telemetry-conventions.md](06-observability/telemetry-conventions.md) (Follow-up questions) | What metric-series and retention budgets apply at deployment scale? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-093 | [telemetry-conventions.md](06-observability/telemetry-conventions.md) (Follow-up questions) | Which collector/backends and tail-sampling capabilities are available? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-094 | [telemetry-conventions.md](06-observability/telemetry-conventions.md) (Follow-up questions) | What stable error taxonomy and protected pseudonymization mechanism are used? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-095 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Follow-up questions) | What incident-mode sampling override and authorization are required? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-096 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Follow-up questions) | Which source coordinates may appear in protected diagnostics? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-097 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Follow-up questions) | What final stable error taxonomy and log event schemas are required? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-098 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Follow-up questions) | Which telemetry fields need pseudonymization? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-099 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Follow-up questions) | What trace and log retention applies relative to metrics and DLQ custody? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-100 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Questions to stamp) | What sampling policy changes during incidents? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-101 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Questions to stamp) | Which stable error taxonomy is shared by logs, metrics, and DLQ? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-102 | [tracing-and-logging.md](06-observability/tracing-and-logging.md) (Questions to stamp) | What payload fields, if any, may appear in operator telemetry? | blocking-milestone | Observability | Project maintainer | open | Observability and diagnostics |
| Q-103 | [deployment-and-orchestration.md](07-operations/deployment-and-orchestration.md) (Follow-up questions) | Which deployment mechanism will invoke migration Jobs? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-104 | [deployment-and-orchestration.md](07-operations/deployment-and-orchestration.md) (Follow-up questions) | Should maintenance Jobs use a separate namespace, node pool, or resource quota? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-105 | [deployment-and-orchestration.md](07-operations/deployment-and-orchestration.md) (Follow-up questions) | What rollout timeout and graceful-drain period fit production partitions? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-106 | [deployment-and-orchestration.md](07-operations/deployment-and-orchestration.md) (Follow-up questions) | Which operations require explicit human approval before execution or target retirement? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-107 | [deployment-and-orchestration.md](07-operations/deployment-and-orchestration.md) (Follow-up questions) | At what migration volume should a dedicated controller be introduced? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-108 | [modes-and-configuration.md](07-operations/modes-and-configuration.md) (Follow-up questions) | Which Kubernetes policy mechanism will restrict each executable and workload? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-109 | [modes-and-configuration.md](07-operations/modes-and-configuration.md) (Follow-up questions) | When should the initial all-mode image be split into separate images? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-110 | [modes-and-configuration.md](07-operations/modes-and-configuration.md) (Follow-up questions) | What exact operational settings may be overridden by flags versus environment variables? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-111 | [modes-and-configuration.md](07-operations/modes-and-configuration.md) (Follow-up questions) | Should migration orchestration eventually become a dedicated controller? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-112 | [runbooks-and-intervention.md](07-operations/runbooks-and-intervention.md) (Follow-up questions) | Which actual teams or rotations fill the role-based ownership categories? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-113 | [runbooks-and-intervention.md](07-operations/runbooks-and-intervention.md) (Follow-up questions) | Where is the canonical incident and evidence system? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-114 | [runbooks-and-intervention.md](07-operations/runbooks-and-intervention.md) (Follow-up questions) | Which commands are supported operator interfaces rather than ad-hoc backend or Kubernetes actions? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-115 | [runbooks-and-intervention.md](07-operations/runbooks-and-intervention.md) (Follow-up questions) | What game-day cadence is appropriate: per release, quarterly, or risk-triggered? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-116 | [runbooks-and-intervention.md](07-operations/runbooks-and-intervention.md) (Follow-up questions) | Which runbooks must be complete before first production deployment? | blocking-milestone | Operations | Project maintainer | open | Operations and deployment |
| Q-117 | [correctness-invariants.md](08-validation/correctness-invariants.md) (Follow-up questions) | Which shared metadata failures justify stopping all projections immediately? | blocking-v1 | Validation and release | Project maintainer | open | Validation and release |
| Q-118 | [correctness-invariants.md](08-validation/correctness-invariants.md) (Follow-up questions) | How are unresolved `unknown` comparisons aged, alerted, and eventually resolved? | blocking-v1 | Validation and release | Project maintainer | open | Validation and release |
| Q-119 | [correctness-invariants.md](08-validation/correctness-invariants.md) (Follow-up questions) | Which convergence preconditions are release-blocking versus operational warnings? | blocking-v1 | Validation and release | Project maintainer | open | Validation and release |
| Q-120 | [source-conflicts.md](source-conflicts.md) (Follow-up questions) | Which relation types are allowed to use source-of-truth fallback, and what exact rate, timeout, and concurrency budgets do they receive? | blocking-milestone | Architecture baseline | Project maintainer | open | Source conflict resolution |
| Q-121 | [source-conflicts.md](source-conflicts.md) (Follow-up questions) | How are fallback reads and failures surfaced in metrics, traces, and diagnostics without exposing protected identifiers? | blocking-milestone | Architecture baseline | Project maintainer | open | Source conflict resolution |

## Standing deferred items

These are accepted decisions' explicit future topics even when they are not phrased as question bullets in a concept document.

| ID | Source | Question / scope | Class | Gate | Owner | Status | Resolution / evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FU-101 | ADR-0033/0034 | Recurring resilience and chaos campaigns | deferred | Post-v1 hardening | Project maintainer | deferred | Safe targeted failure tests remain required |
| FU-102 | ADR-0022 | Live encryption-key rotation | deferred | Security evolution | Project maintainer | deferred | Restart workers for rotation; revisit if operational need changes |
| FU-103 | ADR-0032 | Exact global-stop threshold for shared metadata failures | deferred | Operations evolution | Project maintainer | deferred | Narrowest-safe-scope rule applies meanwhile |
| FU-104 | ADR-0022 | Exact calendar retention and legal treatment of fence identifiers | deferred | Privacy evolution | Project maintainer | deferred | Current retention relationships and DPO ownership apply |
| FU-105 | ADR-0028/0029 | Separate images and a permanent migration controller | deferred | Platform evolution | Project maintainer | deferred | One image and explicit Jobs remain the initial approach |

## Milestone gate

Before starting a milestone, filter the exact-question and standing-deferred
tables by their gate. The milestone is
`ready` only when no applicable item is `open` or `in-progress` in a blocking
class. At completion, every applicable item is `resolved` or explicitly
re-scoped with a recorded safe boundary.

## Related documents

- [Implementation roadmap](../implementation-plan.md)
- [Architecture review method](review-method.md)
- [Decision register](decisions/index.md)
