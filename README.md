# Go CDC Projection Engine

Design-stage repository for a configuration-driven Go engine that projects
MongoDB source-of-truth data, delivered through Kafka CDC streams, into
Elasticsearch read models with Redis-backed operational lookups.

No implementation or source-code architecture has been selected yet.

## Documentation

- [Agent instructions](AGENTS.md) — project-wide workflow and architecture
  guardrails for AI agents and other automated contributors.
- [Architecture review workspace](docs/architecture/index.md) — the design split
  into small, linked decisions to review and stamp one by one.
- [Implementation roadmap](docs/implementation-plan.md) — the gated path from
  the accepted architecture baseline to the first production release.
- [Architecture decision review skill](.skills/architecture-decision-review/SKILL.md)
  — the interactive framework used to review and record those decisions.
- [Original system design draft](docs/design/system.md) — preserved source draft.
- [Original OpenTelemetry draft](docs/design/otel.md) — preserved source draft.

## Repository status

The architecture baseline review is complete. The initial baseline snapshot is
commit `699378f` (2026-09-01); accepted clarifications ADR-0045 and ADR-0046
resolve the baseline-gated fallback observability and eligibility questions.
ADR-0047 resolves Q-001, ADR-0005 resolves Q-002, ADR-0048 resolves Q-003, and
ADR-0049 resolves Q-004. ADR-0050 resolves Q-005 with a 30-day deletion-fence
retention policy and accepted residual expiry risk; ADR-0051 resolves Q-006 with
complete source-bounded child enumeration and a fail-closed handoff. Q-010 is
deferred with snapshot-read events sent to the durable DLQ. The accepted
contract decisions are ready for the first vertical-slice design; the example
projection fixture remains to be selected. ADR-0052 resolves Q-007's required ordering tuple
and comparison scope, ADR-0053 resolves Q-008's canonical typed ID format,
ADR-0054 resolves Q-009's equal-revision handling, ADR-0055 resolves Q-011's
derived Elasticsearch fence fields, and ADR-0056 resolves Q-012's capacity
ownership and enforcement boundary. ADR-0057 resolves Q-013 by defining
reverse-indexed recomputation for mutable references; ADR-0058 resolves Q-014
with benchmark-derived per-relation live fan-out ceilings. ADR-0059 resolves
Q-015 with explicit snapshot, owned-child, and independent-entity lifecycle
roles, separate from physical storage. ADR-0060 resolves Q-016 with a
deterministic data-only Bloblang allowlist and benchmark-evidenced resource
budgets. ADR-0061 resolves Q-017 with entity/relation-level invalidation through
the declared dependency graph; field-level filtering is not required in v1.
ADR-0062 resolves Q-018 by pinning the manifest hash and transformation identity
per physical target; semantic changes use blue-green migration, and no public
per-document version field is required. ADR-0063 resolves Q-126: a
referenced-entity delete recomputes dependent roots, removes optional copied
fields or applies explicit safe missing-value behavior, and never cascades root
deletion. ADR-0064 resolves Q-124: derived fence entity keys and mutation
fingerprints use domain-separated HMAC-SHA-256 tokens, with a target-pinned key
ID and fixed encoding; tokens remain pseudonymous. Q-122 is deferred until the
first fallback-enabled relation. Identify a representative lookup during Step
3a and measure source capacity before enabling fallback; keep it disabled until
the resulting limits are recorded. Q-076 still governs legal retention of fence
identifiers. Q-123 remains an evidence
gate before enabling a production source, and Q-070 supplies numeric capacity
evidence, including Bloblang runtime caps. Remaining blocking follow-ups are
listed in the follow-up register.

## Local prerequisites

None at this stage. Go module metadata, build tooling, dependency choices, and
runtime manifests will be added only after the corresponding architecture
decisions are accepted.
