# Quality attributes

Stamp measurable non-functional requirements and their trade-offs.

- [Performance and capacity](performance-and-capacity.md) - Throughput, latency,
  cardinality, and resource limits; workload-specific objectives and benchmark-
  gated budgets are accepted in ADR-0020; oversized event diagnostics and
  durable custody are specified in
  [ADR-0048](../decisions/0048-event-size-and-buffer-diagnostics.md), while
  reference fan-out uses benchmark-derived live ceilings under ADR-0058.
- [Availability and scaling](availability-and-scaling.md) - Failure domains,
  replicas, rebalances, and dependency degradation; separated health domains,
  workload-group isolation, and multi-signal scaling are accepted in ADR-0021.
- [Security and privacy](security-and-privacy.md) - Access, secrets, sensitive
  data, and deletion obligations; explicit classifications, protected
  diagnostics, source-driven anonymization, and retention-aware expiry are
  accepted in ADR-0022.
- [Compatibility and dependencies](compatibility-and-dependencies.md) - Supported
  versions and upgrade policy; capability-tested matrices, pinned runtimes, and
  guarded upgrades are accepted in ADR-0023.
