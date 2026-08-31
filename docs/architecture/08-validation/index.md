# Validation

Stamp the evidence required before architecture choices and releases are trusted.

- [Correctness invariants](correctness-invariants.md) - Properties that must hold
  for every valid event history; strict safety invariants, conditional
  convergence, explicit `unknown` comparisons, and narrowest-safe-scope failure
  handling are accepted in
  [ADR-0032](../decisions/0032-correctness-invariants.md).
- [Test strategy](test-strategy.md) - Unit, contract, integration, and end-to-end
  coverage; fast per-change, real-dependency integration, and release-level
  end-to-end evidence are accepted in
  [ADR-0033](../decisions/0033-test-strategy.md), while scheduled resilience
  testing remains deferred.
- [Performance and failure testing](performance-and-failure-testing.md) - Load,
  soak, rebalance, and chaos evidence; representative workload benchmarks and
  targeted failure tests are accepted in
  [ADR-0034](../decisions/0034-performance-and-failure-testing.md), while
  recurring resilience campaigns remain deferred.
- [Release acceptance](release-acceptance.md) - Promotion gates and rollback
  evidence; separate gates for binaries, manifests, and migrations, plus an
  audited emergency path, are accepted in
  [ADR-0035](../decisions/0035-release-acceptance.md).
