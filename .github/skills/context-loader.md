---
name: Schema Context Loader
description: 'Fetches exact schema, column descriptions, and metadata for database tables to enable context-aware code generation.'
applyTo: "**/*.sql, **/*.sqlx, **/*.py, **/*.ts, **/*.js"
---

# Schema Context Loader Skill

## Instructions
When working with a specific table, dataset, or database, load its schema metadata before generating or modifying queries. This ensures generated code respects column types, partitioning, and business semantics.

## Process
1. Run the appropriate metadata query for your database engine (see examples below).
2. Analyze column descriptions to understand business logic and intended usage.
3. Use partitioning/clustering metadata to ensure generated SQL includes correct `WHERE` filters for performance.
4. Use nullable/required flags to inform validation logic.

## BigQuery Example
```sql
SELECT
  table_name,
  column_name,
  data_type,
  is_nullable,
  description AS column_description,
  is_partitioning_column,
  clustering_ordinal_position
FROM
  `project`.dataset.INFORMATION_SCHEMA.COLUMNS
WHERE
  table_name IN ('target_table_1', 'target_table_2')
ORDER BY
  table_name, ordinal_position;
```

## PostgreSQL / Cloud SQL Example
```sql
SELECT
  c.table_name,
  c.column_name,
  c.data_type,
  c.is_nullable,
  pgd.description AS column_description
FROM
  information_schema.columns c
LEFT JOIN pg_catalog.pg_description pgd
  ON pgd.objsubid = c.ordinal_position
  AND pgd.objoid = (SELECT oid FROM pg_class WHERE relname = c.table_name)
WHERE
  c.table_schema = 'public'
  AND c.table_name IN ('target_table_1', 'target_table_2')
ORDER BY
  c.table_name, c.ordinal_position;
```

## MySQL Example
```sql
SELECT
  TABLE_NAME,
  COLUMN_NAME,
  DATA_TYPE,
  IS_NULLABLE,
  COLUMN_COMMENT AS column_description
FROM
  INFORMATION_SCHEMA.COLUMNS
WHERE
  TABLE_SCHEMA = 'your_database'
  AND TABLE_NAME IN ('target_table_1', 'target_table_2')
ORDER BY
  TABLE_NAME, ORDINAL_POSITION;
```

## Usage Notes
- Replace placeholder project/dataset/table names with actual values from the project's configuration.
- Column descriptions often encode business rules — read them before generating logic.
- Partitioning columns should always appear in query filters for cost-effective queries.
