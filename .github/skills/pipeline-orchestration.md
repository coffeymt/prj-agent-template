---
name: Pipeline Orchestration
description: 'Orchestration patterns for multi-step data and application pipelines using cloud workflow engines, event triggers, and messaging.'
applyTo: "**/*.yaml, **/*.yml, **/*.json"
---

# Pipeline Orchestration Skill

## Overview
Patterns and best practices for orchestrating multi-step pipelines using cloud workflow engines (Cloud Workflows, Step Functions, Azure Logic Apps), event triggers (Eventarc, EventBridge), and messaging (Pub/Sub, SQS/SNS, Event Hubs).

## Pipeline Phase Pattern
Most data and application pipelines follow a similar phase structure:

### 1. Input Preparation
- **Trigger**: Scheduled (cron), event-driven (file arrival, message), or manual.
- **Action**: Validate inputs, transform raw data into a processing-ready format, export to staging.
- **Output**: Staging data ready for processing + notification to downstream consumers.

### 2. Processing
- **Actor**: Compute resource (VM, container, serverless function, managed service).
- **Action**: Execute core business logic (transformations, ML inference, external API calls, matching, etc.).
- **Output**: Processed results written to storage + completion signal.

### 3. Output Ingestion
- **Trigger**: Event-driven (completion signal via messaging or file arrival).
- **Action**: Ingest processed results, run post-processing (aggregation, historization, dimension updates).
- **Output**: Final tables/views/files ready for consumption.

## Orchestration Best Practices

### Idempotency
- Every pipeline step should be safe to retry. Use unique run IDs as binding keys.
- Store processing state so re-runs skip already-completed work.

### Binding Keys
- Use a `run_id` (or equivalent unique identifier) as the binding key across all phases.
- Pass it through unchanged from input → processing → output.
- All intermediate artifacts should be namespaced by run ID.

### Error Handling
- Define retry policies for transient failures (network timeouts, rate limits).
- Use dead-letter queues/topics for messages that fail after max retries.
- Log failures with enough context to reproduce (run ID, input, error message, timestamp).

### Metadata Validation
- Validate row counts, schema compatibility, and key constraints between pipeline steps.
- Fail fast if upstream data is missing or corrupt — don't propagate bad data downstream.

## Event-Driven Patterns
- **File arrival → trigger**: Use event triggers (Eventarc, EventBridge) to start pipelines when files land in storage.
- **Pub/Sub fan-out**: Publish completion events to a topic; downstream subscribers process independently.
- **Subscription filters**: Use message attribute filters to route events to the correct handler.
- **Message attributes**: Pass all routing metadata (run ID, client ID, source, destination) as message attributes — not in the payload body.

## Workflow ↔ Execution Engine Contract
When a workflow triggers an execution engine (Dataform, dbt, Spark, etc.):
- **Variables must be set at compile/configuration time**, not at execution/invocation time.
- **Targets must use the engine's own naming** (schema + name in the config block, not the underlying dataset).
- **Dependency flags** (transitive dependencies included, transitive dependents included) must be set correctly for the intended scope.
- Always verify the contract between the workflow YAML and the execution engine's configuration before deployment.

## Debugging Tips
- Check workflow execution logs for step-level failures.
- Verify event trigger configurations match the expected event types and filters.
- Confirm message attributes are complete and correctly typed (most messaging systems require string attributes).
- Validate that service accounts have the required permissions for every resource accessed by every step.
