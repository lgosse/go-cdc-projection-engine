---
type: Architecture Review Topic
title: Modes and configuration
description: Defines operational modes, configuration sources, validation, and safeguards.
tags: [operations, cli, configuration]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Modes and configuration

## Decision to stamp

Define the supported operator interface without prematurely freezing exact CLI
flags.

## Draft proposal

Support distinct stream, bootstrap, audit, repair, migration, and DLQ-inspection/
replay operations. Use explicit subcommands or equivalent mode separation, a
documented configuration precedence order, dry-run and validation commands, and
mandatory target/environment confirmation for mutating maintenance operations.

## Pros

- Separates long-running service behavior from bounded jobs.
- Dry-run and validation reduce operational mistakes.
- A shared configuration model prevents mode-specific semantic drift.

## Cons and risks

- One binary accumulates broad credentials and operational surface area.
- CLI compatibility becomes a release concern.
- Some migrations may be safer in a dedicated controller.

## Questions to stamp

- Which operations belong in the binary versus deployment automation?
- What is the precedence among files, environment, and flags?
- Which commands require approval or an exclusive lease?
