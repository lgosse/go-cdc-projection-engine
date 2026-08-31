# System Requirements & Implementation Specification: Go CDC Projection Engine (`gocdc-projector`)

---

## 1. System Objectives & Guarantees

The `gocdc-projector` engine is a stateless, configuration-driven streaming and batch service written in Go. It denormalizes, transforms, and indexes data from isolated MongoDB microservice collections into Elasticsearch compound indices to support low-latency filtering, sorting, and pagination across distributed entities.

### Core Guarantees

* **Zero Downtime Migrations:** Incompatible schema and analyzer updates execute via blue/green indices with **Dual-Write Routing** and atomic alias swaps.
* **Order-Agnostic & Commutative Processing:** Events arriving out of order (e.g., child before parent, stale updates overtaking recent updates) resolve deterministically via field-level timestamp fencing and scripted skeleton upserts.
* **Zero Cross-Database Point Queries at Runtime:** All secondary reference entities and multi-hop foreign-key lookups resolve exclusively against a low-latency Redis cache layer.
* **Domain-Agnostic Core:** The Go binary contains zero domain-specific schemas or hardcoded business logic. All relationships, mappings, and derived calculations are declared via YAML manifests and embedded Bloblang scripts.

---

## 2. Declarative Manifest Specification

Every search index is declared in a single YAML file loaded by the engine at runtime.

### 2.1 Complete Manifest Schema (`manifest.v1.schema.json`)

```yaml
index_name: tasks
version: 2

elasticsearch:
  target_index: tasks_v2
  search_alias: tasks_search
  write_alias: tasks_write
  settings:
    number_of_shards: 3
    number_of_replicas: 1
    refresh_interval: "1s"
  mappings:
    properties:
      task_id: { type: keyword }
      title: { type: text, analyzer: standard }
      status: { type: keyword }
      organisation_id: { type: keyword }
      org_tier: { type: keyword }
      org_country: { type: keyword }
      created_at: { type: date }
      updated_at: { type: date }
      shifts:
        type: nested
        properties:
          shift_id: { type: keyword }
          start_date: { type: date }
          end_date: { type: date }
          status: { type: keyword }
          updated_at: { type: date }
      attendances:
        type: nested
        properties:
          attendance_id: { type: keyword }
          raw_status: { type: keyword }
          computed_status: { type: keyword }
          updated_at: { type: date }
          dispute:
            properties:
              dispute_id: { type: keyword }
              status: { type: keyword }
              updated_at: { type: date }

root_entity:
  topic: cdc.task_service.tasks
  database_uri_env: MONGO_TASK_SERVICE_URI
  database: task_service
  collection: tasks
  id_field: _id
  updated_at_field: updated_at
  fields:
    - { source: _id, target: task_id, type: string }
    - { source: title, target: title, type: string }
    - { source: status, target: status, type: string }
    - { source: organisation_id, target: organisation_id, type: string }
    - { source: created_at, target: created_at, type: date }
    - { source: updated_at, target: updated_at, type: date }

cached_lookups:
  - name: organisation
    topic: cdc.org_service.organisations
    database_uri_env: MONGO_ORG_SERVICE_URI
    database: org_service
    collection: organisations
    lookup_key: organisation_id
    redis_key_prefix: "cache:org:"
    ttl: 604800s # 7 days
    fields:
      - { source: tier, target: org_tier }
      - { source: country, target: org_country }

child_relations:
  - name: shifts
    topic: cdc.shift_service.shifts
    database_uri_env: MONGO_SHIFT_SERVICE_URI
    database: shift_service
    collection: shifts
    join_type: nested_array
    parent_foreign_key: task_id
    child_id_field: _id
    target_field: shifts
    fields:
      - { source: _id, target: shift_id }
      - { source: start_date, target: start_date }
      - { source: end_date, target: end_date }
      - { source: status, target: status }
      - { source: updated_at, target: updated_at }

  - name: attendances
    topic: cdc.attendance_service.attendances
    database_uri_env: MONGO_ATTENDANCE_SERVICE_URI
    database: attendance_service
    collection: attendances
    join_type: nested_array
    parent_foreign_key: task_id
    child_id_field: _id
    target_field: attendances
    fields:
      - { source: _id, target: attendance_id }
      - { source: status, target: raw_status }
      - { source: updated_at, target: updated_at }

  - name: disputes
    topic: cdc.dispute_service.disputes
    database_uri_env: MONGO_DISPUTE_SERVICE_URI
    database: dispute_service
    collection: disputes
    join_type: nested_object
    lookup_parent_key:
      via_cache: "cache:att_to_task:"
      key_field: attendance_id
    child_id_field: _id
    target_field: attendances.dispute
    fields:
      - { source: _id, target: dispute_id }
      - { source: status, target: status }
      - { source: updated_at, target: updated_at }

transformations:
  - scope: "attendances.*"
    target: "computed_status"
    bloblang: |
      root = match {
        this.dispute.status == "PENDING_REVIEW" => "UNDER_DISPUTE",
        this.dispute.status == "RESOLVED_EXCUSED" => "EXCUSED_ABSENCE",
        this.raw_status == "ABSENT" => "UNEXCUSED_ABSENCE",
        _ => this.raw_status | "UNKNOWN"
      }

```

---

## 3. Storage & State Management Specifications

### 3.1 MongoDB Data Layer

* **Prerequisites:** MongoDB 4.4+ Replica Set with Change Streams enabled.
* **Read Preferences:** Batch bootstrap readers must use `secondaryPreferred` to isolate operational primary nodes from analytical full-table scans.

### 3.2 Kafka Streaming Layer

* **Topics & Partitioning:** Topics are partitioned by each source service's native primary key (e.g., `shifts` topic keyed by `shift_id`).
* **Ordering Assumption:** The engine operates under the assumption of **zero partition ordering guarantees** across topics.
* **Consumer Group Configuration:**
* `group.id`: `gocdc-projectors`
* `partition.assignment.strategy`: `cooperative-sticky`
* `enable.auto.commit`: `false` (Manual commit post-Elasticsearch flush)
* `auto.offset.reset`: `earliest`



### 3.3 Redis Caching & Reverse Lookup Layer

Redis is used strictly as a non-persistent operational lookup cache:

| Key Template                              | Data Structure                   | TTL       | Purpose                                                                  |
| ----------------------------------------- | -------------------------------- | --------- | ------------------------------------------------------------------------ |
| `cache:org:<org_id>`                      | String (JSON) or Hash            | 7 Days    | Resolves 1:N reference fields without querying `org_service` DB.         |
| `cache:att_to_task:<att_id>`              | String (`task_id`)               | 30 Days   | Resolves 2-hop reverse foreign key: `dispute.attendance_id -> task_id`.  |
| `cache:active_write_indices:<index_name>` | Set (`["tasks_v2", "tasks_v3"]`) | None      | Dynamic dual-write destination list loaded by streaming workers.         |
| `tombstone:<entity>:<id>`                 | String (`"1"`)                   | 5 Minutes | Prevents delayed out-of-order updates from resurrecting deleted records. |

### 3.4 Elasticsearch Alias & Index Topology

```
                   ┌────────────────────────────────────────┐
                   │               Read Layer               │
                   │         (Frontend / API Gateway)       │
                   └───────────────────┬────────────────────┘
                                       │
                                       ▼ Queries
                           ┌───────────────────────┐
                           │  tasks_search (Alias) │
                           └───────────┬───────────┘
                                       │ Points to Active Version
                                       ▼
                           ┌───────────────────────┐
                           │       tasks_v2        │ (Physical Index)
                           └───────────▲───────────┘
                                       │
                ┌──────────────────────┴──────────────────────┐
                │ Dual-Write HTTP _bulk                       │ Dual-Write HTTP _bulk
                │                                             │
┌───────────────┴───────────────┐             ┌───────────────┴───────────────┐
│     gocdc-projector Pod 1     │             │     gocdc-projector Pod N     │
└───────────────────────────────┘             └───────────────────────────────┘
                                                              │
                                                              │ Dual-Write (During Migration)
                                                              ▼
                                                  ┌───────────────────────┐
                                                  │       tasks_v3        │ (Target Index)
                                                  └───────────────────────┘

```

* Every physical index follows the naming convention `<index_name>_v<version>`.
* `tasks_search`: Points to exactly **one** active physical index.
* `tasks_write`: Reserved as an operational alias pointing to the primary write index. During dual-write, the Go worker overrides single-alias writes by dynamically fetching all target indices from `cache:active_write_indices:<index_name>`.

---

## 4. Component & Pipeline Specifications

### 4.1 Worker Boot Sequence & Startup Handshake

Upon container boot, the Go process executes the following deterministic lifecycle:

```
[ Container Boot ]
        │
        ├── 1. Load all manifests from /etc/projector/manifests/*.yaml
        │
        ├── 2. Compile Bloblang transformation ASTs & Painless script templates into memory
        │
        ├── 3. Redis & Kafka Connectivity Verification
        │
        ├── 4. Schema Handshake with Elasticsearch:
        │      FOR EACH manifest:
        │        • Query ES for physical indices behind `search_alias` and `cache:active_write_indices`
        │        • Fetch live index mappings via GET /<index>/_mapping
        │        • Compare live mapping vs. YAML mapping:
        │            - IF IDENTICAL: Pass validation.
        │            - IF ADDITIVE (New fields only): Issue PUT /<index>/_mapping. Pass validation.
        │            - IF INCOMPATIBLE (Field type change):
        │                * Check if active_write_indices contains migration target (e.g., tasks_v3)
        │                * IF YES: Attach dual-write sink. Pass validation.
        │                * IF NO: Flag index as BLOCKED_SCHEMA_MISMATCH.
        │
        ├── 5. Subscribe to Kafka Topics (Cooperative Sticky Assignor)
        │      • IF any index is BLOCKED_SCHEMA_MISMATCH:
        │          Call consumer.Pause([TopicPartition]) for topics associated with blocked indices.
        │
        ├── 6. Launch Ticker (10s interval) to poll ES/Redis for unblocking paused partitions
        │
        ├── 7. Spawn Worker Goroutine Pools & In-Memory Coalescers
        │
        └── 8. Start HTTP Server (:8080) -> /healthz (200), /readyz (200 if no fatal errors)

```

---

### 4.2 In-Memory Coalescing & Out-of-Order CDC Pipeline

To handle unsorted Kafka traffic with high write efficiency, incoming events pass through an in-memory batch coalescer before hitting Elasticsearch.

```
[ Kafka Consumer Loop ]
           │
           ├── Read batch of up to 1,000 messages (or 500ms timeout)
           │
           ▼
[ In-Memory Coalescer Map: map[TargetKey][]CDCEvent ] (TargetKey = index_name + doc_id)
           │
           ├── Group events by target document ID (task_id)
           │
           ├── Extract distinct lookup keys (e.g., org_ids) -> Issue 1 Pipeline Redis MGET
           │
           ├── Execute Bloblang transformations in Go memory on intermediate payloads
           │
           ▼
[ Elasticsearch _bulk Payload Generator ]
           │
           ├── Generates EXACTLY 1 composite Painless Scripted Upsert per unique doc_id
           │
           ├── Multiplies action across all active indices in `cache:active_write_indices`
           │
           ▼
[ HTTP Client POST /_bulk ] (retry_on_conflict: 5)
           │
           ├── On Success: Commit Kafka offsets for batch
           └── On Partial 4xx/5xx: Extract poison pills -> Route to DLQ -> Commit remaining offsets

```

#### Complete Painless Script Template for Scripted Upsert (`tasks`)

This script handles root updates, nested array upserts, and out-of-order sub-entity timestamp fencing:

```painless
// 1. Root Entity Upsert Logic
if (params.root_update != null) {
  if (ctx._source.updated_at == null || ctx._source.updated_at <= params.root_update.updated_at) {
    ctx._source.task_id = params.root_update.task_id;
    ctx._source.title = params.root_update.title;
    ctx._source.status = params.root_update.status;
    ctx._source.organisation_id = params.root_update.organisation_id;
    ctx._source.org_tier = params.root_update.org_tier;
    ctx._source.org_country = params.root_update.org_country;
    ctx._source.created_at = params.root_update.created_at;
    ctx._source.updated_at = params.root_update.updated_at;
  }
}

// 2. Child Entity Upsert Logic (Shifts Array)
if (params.shift_update != null) {
  if (ctx._source.shifts == null) {
    ctx._source.shifts = new ArrayList();
  }
  
  def targetShift = null;
  for (item in ctx._source.shifts) {
    if (item.shift_id == params.shift_update.shift_id) {
      targetShift = item;
      break;
    }
  }
  
  if (targetShift == null) {
    ctx._source.shifts.add(params.shift_update);
  } else {
    // Timestamp fencing on shift entity
    if (targetShift.updated_at == null || targetShift.updated_at <= params.shift_update.updated_at) {
      targetShift.putAll(params.shift_update);
    }
  }
}

// 3. Nested Grandchild Upsert Logic (Attendances & Disputes)
if (params.attendance_update != null) {
  if (ctx._source.attendances == null) {
    ctx._source.attendances = new ArrayList();
  }
  
  def targetAtt = null;
  for (item in ctx._source.attendances) {
    if (item.attendance_id == params.attendance_update.attendance_id) {
      targetAtt = item;
      break;
    }
  }
  
  if (targetAtt == null) {
    targetAtt = ["attendance_id": params.attendance_update.attendance_id];
    ctx._source.attendances.add(targetAtt);
  }
  
  if (targetAtt.updated_at == null || targetAtt.updated_at <= params.attendance_update.updated_at) {
    targetAtt.raw_status = params.attendance_update.raw_status;
    targetAtt.updated_at = params.attendance_update.updated_at;
    targetAtt.computed_status = params.attendance_update.computed_status;
    if (params.attendance_update.dispute != null) {
      targetAtt.dispute = params.attendance_update.dispute;
    }
  }
}

```

---

### 4.3 Bootstrap & Cold-Start Batch Engine

The bootstrap mode (`./projector --mode=bootstrap`) runs as a multi-threaded parallel job.

```
[ CLI: ./projector --mode=bootstrap --manifest=tasks.yaml --target-index=tasks_v3 --workers=8 ]
                                       │
                                       ▼
                [ Step 1: Range Partitioning Coordinator ]
                Queries MongoDB `tasks` collection for min(_id) and max(_id).
                Splits ID space into N equal chunks using cursor range boundaries.
                                       │
                                       ▼
                ┌────────────────────────────────────────────────────────┐
                │ Worker Pool (8 Parallel Worker Goroutines)             │
                │                                                        │
                │ For each Chunk [_id_min to _id_max]:                   │
                │  1. Read 2,000 Task documents from Mongo Secondary.   │
                │  2. Extract distinct task_ids and organisation_ids.    │
                │  3. Execute Redis MGET for organisation metadata.      │
                │  4. Execute MongoDB $in queries:                       │
                │     • db.shifts.find({ task_id: { $in: task_ids } })   │
                │     • db.attendances.find({ task_id: { $in: task_ids }│
                │     • db.disputes.find({ attendance_id: { $in: ... } })│
                │  5. Join entities in Go memory into unified structs.   │
                │  6. Execute Bloblang transformation AST in memory.     │
                │  7. Pipeline Redis MSET:                               │
                │     cache:att_to_task:<att_id> -> task_id              │
                │  8. Dispatch ES _bulk index request to tasks_v3.       │
                └──────────────────────────┬─────────────────────────────┘
                                           │
                                           ▼
                [ Step 2: Signal Completion & Exit 0 ]

```

---

### 4.4 Dual-Write Blue/Green Migration Engine

When an incompatible schema modification occurs (e.g., manifest version increments from `2` to `3`), the migration is orchestrated end-to-end via an automated CI/CD pipeline or Helm Pre-Upgrade Hook.

```
MIGRATION TIMELINE & DUAL-WRITE LIFECYCLE

Time ─────────────────────────────────────────────────────────────────────────────────────────────────────────►

1. Provision Target Index
   • PUT /tasks_v3 (with mappings from tasks.yaml v3)
   • SET cache:active_write_indices:tasks = ["tasks_v2", "tasks_v3"]
   
2. Streaming Workers Enter Dual-Write
   • Live CDC workers detect Redis update
   • Every incoming CDC write is duplicated to BOTH tasks_v2 and tasks_v3
   • tasks_search STILL points to tasks_v2 (Zero read disruption)

3. Run Bootstrap Job
   • Launch K8s Job: projector --mode=bootstrap --target-index=tasks_v3 --workers=16
   • Scans historical MongoDB data, runs Bloblang, bulk loads tasks_v3
   • Live CDC updates to tasks_v3 override historical scans via timestamp fencing

4. Verification & Alias Cutover
   • Await Bootstrap Job exit code 0
   • Verify document counts: count(tasks_v3) >= count(tasks_v2)
   • Atomic Alias Swap:
     POST /_aliases
     {
       "actions": [
         { "remove": { "index": "tasks_v2", "alias": "tasks_search" } },
         { "add":    { "index": "tasks_v3", "alias": "tasks_search" } }
       ]
     }
   • Frontend immediately queries tasks_v3 with zero downtime

5. Tear Down Dual-Write & Deprecate
   • SET cache:active_write_indices:tasks = ["tasks_v3"]
   • Workers stop writing to tasks_v2
   • DELETE /tasks_v2

```

---

### 4.5 Automated Drift Detection & Reconciliation Auditor

To safeguard against network drops or unhandled edge cases, a reconciliation auditor runs as a nightly Kubernetes `CronJob` (`./projector --mode=audit`).

```
1. Sample Selection:
   Query MongoDB `tasks` collection for 1% random sample of IDs updated in past 7 days:
   db.tasks.aggregate([
     { $match: { updated_at: { $gte: ISODate("NOW - 7 DAYS") } } },
     { $sample: { size: 5000 } }
   ])

2. Assemble Ground Truth:
   Fetch shifts, attendances, disputes from Mongo + org metadata from Redis.
   Run Bloblang transformation to produce Canonical Materialized Struct.

3. Fetch Elasticsearch Documents:
   Issue POST /tasks_search/_mget for the 5,000 IDs.

4. Canonical Hash Comparison:
   Compute SHA256 over normalized, key-sorted fields for both payloads:
   Hash_Mongo == Hash_ES ?
     • YES: Record matches.
     • NO:  DRIFT DETECTED.
            - Increment Prometheus metric `gocdc_drift_detected_total{index="tasks"}`.
            - Push task_id to high-priority repair buffer.
            - Re-index task document directly into active write index.

```

---

### 4.6 Error Handling, Retry Policies & Dead Letter Queue (DLQ)

```
[ Error Occurs During Processing ]
                │
                ├── Is it a Transient / Network Error? (ES 429, 502, 503, Redis Timeout)
                │     ├── Retry with Exponential Backoff (100ms, 500ms, 2s)
                │     └── Up to 5 attempts -> If exhausted: Panic Pod (K8s restarts pod)
                │
                └── Is it a Poison Pill / Data Error? (ES 400 Mapping Rejection, Bloblang AST Failure)
                      │
                      ├── 1. Wrap event in DLQ Envelope:
                      │      {
                      │        "index": "tasks",
                      │        "error_type": "MAPPING_CONFLICT",
                      │        "error_message": "...",
                      │        "payload": { ...raw cdc event... },
                      │        "timestamp": 1725000000000
                      │      }
                      │
                      ├── 2. Write envelope to MongoDB `cdc_projection_dlq` collection
                      │
                      ├── 3. Increment metric `gocdc_dlq_events_total{index="tasks"}`
                      │
                      └── 4. Advance Kafka Offset (Pipeline continues for remaining healthy events)

```

---

## 5. Observability & Alerting Matrix

| Metric Name                         | Type      | Target Threshold    | Alert Severity                                    | Action Required                                                    |
| ----------------------------------- | --------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------------------ |
| `gocdc_kafka_consumer_lag_records`  | Gauge     | `< 5000`            | Warning ($>10\text{k}$), Critical ($>50\text{k}$) | Scale worker pods via HPA.                                         |
| `gocdc_bulk_flush_duration_seconds` | Histogram | P99 `< 1.0\text{s}` | Warning (P99 $> 2\text{s}$)                       | Check Elasticsearch disk I/O and indexing queue.                   |
| `gocdc_dlq_events_total`            | Counter   | `0`                 | Critical (Rate $> 0$ for $5\text{m}$)             | Inspect `cdc_projection_dlq` for schema/type mismatches.           |
| `gocdc_drift_detected_total`        | Counter   | `0`                 | Warning ($> 10$ / run)                            | Investigate missing CDC events or timing bugs in Painless scripts. |
| `gocdc_redis_cache_miss_total`      | Counter   | `< 5\%` of lookups  | Warning ($> 15\%$)                                | Check Redis eviction policy and TTL configurations.                |
| `gocdc_schema_mismatch`             | Gauge     | `0`                 | Critical ($= 1$)                                  | Dual-write target index not provisioned; check migration pipeline. |

---

## 6. Codebase Structure & CLI Flags

```
gocdc-projector/
├── cmd/
│   └── projector/
│       └── main.go                  # CLI Entrypoint
├── pkg/
│   ├── config/
│   │   ├── manifest.go              # Manifest loader & YAML parser
│   │   └── validator.go             # Schema & relation graph validator
│   ├── transform/
│   │   ├── bloblang.go              # Embedded Bloblang runtime wrapper
│   │   └── compiler.go              # Pre-compilation of Bloblang ASTs
│   ├── source/
│   │   ├── kafka_consumer.go        # Cooperative-sticky Kafka consumer loop
│   │   └── mongo_scanner.go         # Range-based chunked batch scanner
│   ├── cache/
│   │   ├── redis_client.go          # Pipeline MGET, MSET, & tombstone handlers
│   │   └── state_manager.go         # Active write index synchronization
│   ├── sink/
│   │   ├── coalescer.go             # In-memory document batch coalescer
│   │   ├── painless_builder.go      # Dynamic AST-to-Painless script generator
│   │   └── elastic_bulk.go          # Dual-write Elasticsearch HTTP bulk client
│   ├── migration/
│   │   ├── orchestrator.go          # Blue/Green cutover & validation engine
│   │   └── handshake.go             # Boot-time mapping & alias verifier
│   └── auditor/
│       ├── sampler.go               # MongoDB statistical document sampler
│       └── hasher.go                # Canonical SHA256 hash comparison engine
├── manifests/
│   ├── tasks.yaml
│   ├── users.yaml
│   └── ... (28 other index manifests)
├── Dockerfile
└── Makefile

```

### CLI Command Reference

```bash
# 1. Run live real-time CDC projection worker across all loaded manifests
./projector --mode=stream --manifest-dir=/etc/projector/manifests/

# 2. Run bootstrap job for a single index into a specific physical target
./projector \
  --mode=bootstrap \
  --manifest=/etc/projector/manifests/tasks.yaml \
  --target-index=tasks_v3 \
  --workers=16 \
  --batch-size=2000 \
  --mongo-read-pref=secondaryPreferred

# 3. Run reconciliation audit cron
./projector \
  --mode=audit \
  --manifest=/etc/projector/manifests/tasks.yaml \
  --sample-rate=0.01 \
  --auto-repair=true

# 4. Replay Dead Letter Queue messages
./projector \
  --mode=dlq-replay \
  --manifest=/etc/projector/manifests/tasks.yaml \
  --limit=1000

```

---

## Follow-up Topics for Detailed Discussion

While the overall architecture, data contracts, and dual-write mechanics are specified, the following concrete implementation details should be reviewed next:

1. **Nested Array Identity & Array Sizing Limits in Painless:**
* Determining the maximum number of nested items (e.g., shifts per task) before Elasticsearch nested document limits (`index.mapping.nested_objects.limit: 10000`) or Painless linear iteration overhead degrade search and indexing performance.


2. **Dynamic Bloblang-to-Painless Translation Boundary:**
* Establishing the exact subset of Bloblang operations allowed in manifests to ensure they can be mapped deterministically to Go memory execution (during batch bootstrap) and Painless script execution (during live CDC updates).


3. **Kafka Partition Assignment & Rebalance Tuning:**
* Choosing between a **Single Shared Consumer Group** across all 30 manifests versus **Topic-Grouped Consumer Pools** (e.g., 1 pool for high-throughput indexes, 1 pool for standard indexes) to minimize partition rebalance churn across worker pods during Kubernetes HPA scaling events.


4. **Primary Key Type Standardization (BSON ObjectID vs. UUID Strings):**
* Defining the exact canonical string serialization and hex conversion rules across heterogeneous MongoDB collections when assembling join keys and Elasticsearch `_id` values.


5. **Helm Pre-Upgrade Hook & Orchestration Failure Recovery:**
* Detailed state recovery and rollback procedures if the Bootstrap Job fails midway through a Blue/Green migration (e.g., cleaning up orphaned `_v3` indices and purging invalid entries from `cache:active_write_indices`).