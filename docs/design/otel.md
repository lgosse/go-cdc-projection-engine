# OpenTelemetry Observability Design

To provide complete observability across both streaming and batch modes, the `gocdc-projector` engine should emit OpenTelemetry (OTel) data across four distinct pillars: **Resource Attributes**, **Metrics**, **Traces (Spans)**, and **Structured Logs**.

---

### 1. Resource Attributes & Common Scope

Every metric, span, and log must be enriched with standardized OpenTelemetry semantic resource attributes so you can slice data across all 30 manifests:

```yaml
# Global Resource Attributes
service.name: "gocdc-projector"
service.version: "v1.4.0"
deployment.environment: "production"
k8s.pod.name: "gocdc-tasks-stream-7b89f5d6-x4z9q"

# Custom Engine Scope (Tagged at Tracer/Meter creation)
projection.index: "tasks"
projection.version: "2"
projection.mode: "stream" # "stream" | "bootstrap" | "audit" | "dlq-replay"

```

---

### 2. OpenTelemetry Metrics

#### A. Ingestion & Change Streams (CDC)

* `cdc.events.received.total` (*Counter*): Count of incoming Change Stream events partitioned by `collection` and `operation` (`insert`, `update`, `replace`, `delete`).
* `cdc.consumer.lag.seconds` (*Gauge*): Current time minus the Mongo event `ClusterTime`. Primary alert trigger for pipeline delay.
* `cdc.resume_token.checkpoint.duration.seconds` (*Histogram*): Latency of persisting ChangeStream tokens to Redis.
* `cdc.stream.reconnects.total` (*Counter*): Count of Change Stream disconnects and cursor renewals.

#### B. Batch Scanner (Bootstrap Mode)

* `bootstrap.records.scanned.total` (*Counter*): Total historical documents read from primary collections.
* `bootstrap.chunks.active` (*UpDownCounter*): Number of parallel range cursors currently processing.
* `bootstrap.chunks.completed.total` (*Counter*): Completed MongoDB scan buckets.
* `bootstrap.chunk.duration.seconds` (*Histogram*): Execution time per 2,000-document range scan.

#### C. Redis Cache & Lookups

* `cache.lookups.total` (*Counter*): Lookups by `status` (`hit` vs `miss`) and `lookup_name` (`organisation`, `att_to_task`).
* `cache.lookup.duration.seconds` (*Histogram*): Latency of Redis `MGET` / `GET` calls.
* `cache.fallback_db_reads.total` (*Counter*): Number of times a cache miss forced a direct read to an external MongoDB service.
* `cache.reverse_index.writes.total` (*Counter*): Number of reverse relation entries written (e.g., `att_id -> task_id`).

#### D. Bloblang Transformation Engine

* `bloblang.evaluations.total` (*Counter*): Executions by `scope` (e.g., `attendances.*`).
* `bloblang.execution.duration.seconds` (*Histogram*): Microsecond latency of executing in-memory Bloblang scripts.
* `bloblang.errors.total` (*Counter*): Runtime evaluation failures (e.g., type casting errors, null pointer references).

#### E. Elasticsearch Sink & Bulk Buffer

* `elasticsearch.bulk.requests.total` (*Counter*): Total HTTP `_bulk` calls by `status_code` (`200`, `429`, `500`).
* `elasticsearch.bulk.duration.seconds` (*Histogram*): Latency of Elasticsearch bulk ingestion.
* `elasticsearch.documents.indexed.total` (*Counter*): Individual documents pushed via bulk.
* `elasticsearch.version_fencing.dropped.total` (*Counter*): Documents where Painless set `ctx.op = "none"` because a newer CDC record already updated the index.
* `elasticsearch.bulk.retries.total` (*Counter*): Retried bulk requests due to 429 backpressure or version conflicts.

#### F. Drift Reconciliation & DLQ

* `drift.audited.documents.total` (*Counter*): Total documents sampled by the auditor.
* `drift.discrepancies.detected.total` (*Counter*): Number of hash mismatches found between Mongo and ES.
* `drift.repairs.triggered.total` (*Counter*): Corrective upserts dispatched to ES.
* `dlq.events.written.total` (*Counter*): Poison-pill documents routed to the DLQ collection.

---

### 3. Trace Spans & Distributed Context

Traces give end-to-end visibility into individual event processing paths, database joins, and batch write flushes.

#### Stream Mode Trace Tree (Single Event Lifecycle)

```
[ cdc.process_event ] 
  ├── Attributes: 
  │     collection: "shifts", op: "update", 
  │     parent_id: "task_100", doc_id: "shift_50"
  │
  ├── [ redis.resolve_parent_key ] (If multi-hop lookup needed)
  │     └── Attributes: key: "cache:shift_to_task:shift_50", hit: true
  │
  ├── [ bloblang.transform ]
  │     └── Attributes: scope: "shifts.*", target: "shifts"
  │
  └── [ buffer.enqueue ]
        └── Attributes: buffer_size: 412, buffer_capacity: 1000

```

#### Flush Trace Tree (Elasticsearch Bulk Execution)

```
[ elasticsearch.flush_bulk ]
  ├── Attributes: 
  │     index: "tasks_v2", batch_size: 1000, 
  │     flush_reason: "size_threshold" # or "interval_timer"
  │
  └── [ http.post /_bulk ]
        └── Attributes: 
              http.status_code: 200, took_ms: 45, 
              items.success: 998, items.dropped_version_fence: 2

```

#### Bootstrap Mode Trace Tree (Per Chunk)

```
[ bootstrap.process_chunk ]
  ├── Attributes: 
  │     chunk.id: 14, range_start: "64a000000000", range_end: "64b000000000"
  │
  ├── [ mongo.find_root_batch ]
  │     └── Attributes: batch_size: 2000, duration_ms: 120
  │
  ├── [ mongo.find_children_batch ] (Parallel goroutines per child relation)
  │     ├── [ mongo.find_shifts ] (db.shifts.find({ task_id: { $in: [...] } }))
  │     └── [ mongo.find_attendances ]
  │
  ├── [ redis.mget_metadata ]
  │     └── Attributes: requested_keys: 2000, found_keys: 1985, cache_hit_ratio: 0.9925
  │
  ├── [ bloblang.batch_transform ]
  │     └── Attributes: total_documents: 2000, duration_ms: 4
  │
  └── [ elasticsearch.bulk_insert ]
        └── Attributes: index: "tasks_v2", items_count: 2000

```

---

### 4. Structured Logs with Trace Context

All engine logs should use structured JSON and automatically attach `trace_id` and `span_id` using OpenTelemetry logger bridges:

| Level       | Event Scenario                             | Log Payload Fields                                              |
| ----------- | ------------------------------------------ | --------------------------------------------------------------- |
| **`INFO`**  | Manifest loaded / Index initialized        | `manifest`, `version`, `target_index`, `aliases`                |
| **`INFO`**  | Alias swap complete (Bootstrap finished)   | `old_index`, `new_index`, `search_alias`, `total_docs`          |
| **`WARN`**  | Redis cache miss (Triggered DB fallback)   | `lookup_name`, `missing_key`, `fallback_db`, `duration_ms`      |
| **`WARN`**  | Version fencing dropped stale batch record | `doc_id`, `incoming_updated_at`, `existing_updated_at`          |
| **`WARN`**  | Elasticsearch 429 backpressure received    | `retry_count`, `backoff_duration_ms`, `queue_depth`             |
| **`ERROR`** | Poison pill routed to DLQ                  | `doc_id`, `collection`, `error_reason`, `raw_payload`, `dlq_id` |
| **`ERROR`** | Drift auditor hash mismatch                | `doc_id`, `mongo_hash`, `es_hash`, `mismatched_fields`          |

---

### 5. OpenTelemetry Collector Pipeline Configuration

Route these telemetry signals from the projector engine to your backend (Prometheus/Grafana, Datadog, or Elasticsearch/Kibana) via the standard OTel Collector:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
  otlp/traces:
    endpoint: "tempo:4317"
    tls:
      insecure: true
  elasticsearch/logs:
    endpoints: ["http://elasticsearch:9200"]
    logs_index: "gocdc-logs"

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp/traces]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [elasticsearch/logs]

```