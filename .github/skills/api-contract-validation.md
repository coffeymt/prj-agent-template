---
name: API Contract Validation
description: 'Rules governing the interface between orchestration workflows and execution engines, including variable passing, compilation, and invocation patterns.'
applyTo: "**/*.yaml, **/*.yml, **/*.sql, **/*.sqlx"
---

# API Contract Validation Skill

## Overview
This skill documents best practices for the interface between orchestration layers (workflow engines, CI/CD pipelines, API gateways) and execution engines (Dataform, dbt, Spark, custom services). Misunderstanding these interfaces causes **silent runtime failures** — the orchestrator reports success but the engine produces incorrect results.

## The Compile-Then-Execute Pattern

Many execution engines use a two-phase model:

### Phase 1: Compilation / Configuration
- Source code or templates are compiled into an executable plan.
- **Variables and parameters are resolved HERE** — template expressions, config references, and environment variables are substituted.
- Output: An executable artifact (compiled SQL, execution plan, container image).

### Phase 2: Execution / Invocation
- The compiled artifact is executed against the target system.
- **Variables set at this stage may have NO effect** on template expressions that were already resolved during compilation.
- Output: Results, logs, and status.

## Critical Rule: Variables Must Be Set at the Right Phase

### When to set variables at compile time:
- Template expressions (`${var.name}`, `{{ config.name }}`, `$VARIABLE`)
- Schema/target references that are resolved during compilation
- Feature flags that control which code paths are included

### When to set variables at execution time:
- Runtime parameters passed via API/CLI arguments
- Query parameters in parameterized SQL (e.g., `@param` in BigQuery)
- Environment variables read by the executing process

### Common Mistake
Setting a compile-time variable at execution time. The variable is silently ignored, and the engine uses either:
- A default value (producing wrong but non-failing results)
- No value (causing a compilation error)

## Target & Schema Mapping

When an orchestrator targets specific objects in an execution engine:
- The **target name** must match the engine's internal name (not the underlying storage name).
- The **schema/dataset** must match the engine's configured schema (not the physical dataset).
- Example: if the engine defines `config.schema = "silver"`, the orchestrator must target `schema: "silver"` — not the underlying BigQuery dataset name.

## Dependency Flags

Most engines support dependency inclusion flags:

| Flag | Meaning | Use Case |
|---|---|---|
| Include upstream dependencies | Also execute all prerequisites of the target | Use for "ensure everything is ready" runs |
| Include downstream dependents | Also execute everything that depends on the target | Use for "cascade through the pipeline" runs |
| Target only | Execute only the specified target | Use for isolated operations or testing |

Choose flags carefully — running unnecessary upstream/downstream targets wastes compute and may cause unintended side effects.

## Message & Event Contracts

When orchestration communicates via messaging (Pub/Sub, SQS, etc.):
- Define a strict message attribute schema. All routing metadata should be in attributes, not the message body.
- Document required vs. optional attributes.
- Use the binding key (e.g., `run_id`) consistently across all messages in a pipeline run.
- Validate message completeness before processing — reject incomplete messages to a dead-letter queue.

## Configuration File Contracts

When an orchestrator reads configuration from a file or table:
- Define the schema (required fields, types, valid values).
- Validate configuration at startup — fail fast on invalid config.
- Document every field's purpose and consumers.
- Version the config schema if it evolves over time.

## Checklist for New Orchestration ↔ Engine Integrations

When creating a new workflow that triggers an execution engine:
- [ ] Variables are set at the correct phase (compile vs. execution)
- [ ] Default values exist in configuration for all required variables
- [ ] Target names and schemas match the engine's internal definitions
- [ ] Dependency inclusion flags are set correctly for the intended scope
- [ ] Binding keys (run IDs, correlation IDs) are passed consistently through all phases
- [ ] Error handling covers both orchestration failures and engine failures
- [ ] Retry policies are idempotent — re-running a step produces the same result

When creating a new engine object (table, model, function) that is triggered by an orchestrator:
- [ ] All variables are declared with safe defaults in the engine's configuration
- [ ] The orchestrator passes all required variables at the correct phase
- [ ] Variable values are cast to the correct type before use (orchestrator vars are often strings)
- [ ] The object's name and schema match what the orchestrator references
