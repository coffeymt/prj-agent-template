---
name: SQL Development Standards
description: 'SQL authoring standards, configuration rules, incremental patterns, and dynamic SQL conventions.'
applyTo: "**/*.sql, **/*.sqlx"
---

# SQL Development Skill

## Overview
Standards for developing SQL scripts and managing database objects across projects. Applies to raw SQL, Dataform SQLX, dbt models, and other SQL-based transformation frameworks.

## Configuration Rules
- **Schema/Dataset**: Must be explicit in every object definition. Never rely on implicit defaults.
- **Tags/Labels**: Tag objects with at least their purpose (`input`/`output`/`staging`) and domain (`customer`/`order`/`product`).
- **Descriptions**: Mandatory for all tables, views, and materialized views. Describe what it contains and its role in the pipeline.
- **Partitioning**: Required for transaction/history/event tables. Choose the right time granularity.
- **Clustering/Indexing**: Required for large tables. Optimize for the most common query patterns.

## Referencing & Dependencies
- **Use framework references**: Always use the framework's dependency mechanism (`${ref("table")}` in Dataform, `{{ ref('model') }}` in dbt, etc.) for all table references. This registers dependencies in the DAG.
- **Resolve without dependency**: If you need to reference a table without creating a dependency (e.g., to break circular references), use the framework's resolution function (`${resolve()}` in Dataform, `{{ source() }}` in dbt).
- **Self-references**: Use the framework's self-reference mechanism (`${self()}` in Dataform, `{{ this }}` in dbt) within incremental logic blocks.
- **Never hardcode** project IDs, dataset names, or fully qualified table paths.

## Incremental Logic
- Identify the unique/merge key for the target table.
- Use the standard incremental filtering pattern to process only new or changed records:
  ```sql
  -- Example: Dataform incremental pattern
  ${when(incremental(), `
    WHERE updated_at > (SELECT MAX(updated_at) FROM ${self()})
  `)}
  ```
- For append-only tables (history, audit logs):
  ```sql
  -- Only append records not already present
  WHERE id NOT IN (SELECT id FROM ${self()})
  ```
- Always ensure idempotency — running the same incremental load twice should produce the same result.

## Dynamic SQL
- Use pre-operations or hooks for variable declarations, temp table generation, or conditional logic:
  ```sql
  -- Example: Dynamic source union
  pre_operations {
    DECLARE vSql STRING;
    SET vSql = (SELECT STRING_AGG(...) FROM ...);
    EXECUTE IMMEDIATE "CREATE OR REPLACE TEMP TABLE ... AS " || vSql;
  }
  SELECT * FROM temp_table
  ```
- Keep dynamic SQL readable — comment the intent of each dynamic block.
- Prefer static SQL when the logic is fixed. Use dynamic SQL only when the structure varies at runtime.

## SQL Style
- **Keywords**: UPPERCASE (`SELECT`, `FROM`, `WHERE`, `JOIN`).
- **Identifiers**: lowercase_with_underscores for columns, tables, and aliases.
- **Aliases**: Always alias tables in JOINs. Use meaningful abbreviations, not single letters.
- **Column Lists**: Prefer explicit column lists over `SELECT *`.
- **Comments**: Explain business logic, invariants, and non-obvious `WHERE` clauses. Don't comment obvious syntax.

## Assertions & Testing
- Use inline assertions for uniqueness and non-null checks on key columns.
- Use dedicated assertion/test files for complex cross-table validation (referential integrity, range checks, aggregation tests).
- Every table with a primary/unique key should have a uniqueness assertion.
