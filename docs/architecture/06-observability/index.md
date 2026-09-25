# Observability

Stamp telemetry around decisions and service objectives, not around draft
implementation details.

- [Telemetry conventions](telemetry-conventions.md) - Resource identity,
  attribute policy, and context propagation; bounded dimensions, protected
  diagnostics, sampled traces, and non-blocking export are accepted in ADR-0024;
  fallback-read observability and identifier handling are specified in
  [ADR-0045](../decisions/0045-fallback-read-observability.md).
- [Metrics and alerting](metrics-and-alerting.md) - SLO signals, bounded
  dimensions, and actionable alerts; canonical metric groups and owned,
  symptom-based alerting are accepted in ADR-0025.
- [Tracing and logging](tracing-and-logging.md) - Sampling, event boundaries,
  structured events, and redaction; meaningful boundaries, sampled event spans,
  protected diagnostics, and non-blocking export are accepted in ADR-0026.
- [Health and diagnostics](health-and-diagnostics.md) - Liveness, readiness,
  progress, and operator inspection; stable scoped probes and authenticated
  authoritative diagnostics are accepted in ADR-0027.
