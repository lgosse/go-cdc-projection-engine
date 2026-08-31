# Go CDC Projection Engine

Design-stage repository for a configuration-driven Go engine that projects
MongoDB source-of-truth data, delivered through Kafka CDC streams, into
Elasticsearch read models with Redis-backed operational lookups.

No implementation or source-code architecture has been selected yet.

## Documentation

- [Architecture review workspace](docs/architecture/index.md) — the design split
  into small, linked decisions to review and stamp one by one.
- [Original system design draft](docs/design/system.md) — preserved source draft.
- [Original OpenTelemetry draft](docs/design/otel.md) — preserved source draft.

## Repository status

The repository is in architecture discovery. The material under
`docs/architecture/` is provisional unless a concept is explicitly marked as
accepted.

## Local prerequisites

None at this stage. Go module metadata, build tooling, dependency choices, and
runtime manifests will be added only after the corresponding architecture
decisions are accepted.
