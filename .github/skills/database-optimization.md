---
name: Database Optimization
description: 'Partitioning, indexing, query performance, and cost management best practices for analytical and transactional databases.'
applyTo: "**/*.sql, **/*.sqlx, **/*.py, **/*.ts, **/*.js"
---

# Database Optimization Skill

## Overview
Best practices for database table performance and cost management across analytical (BigQuery, Redshift, Snowflake) and transactional (PostgreSQL, MySQL, Cloud SQL) engines.

## Partitioning
- **Time-Unit Partitioning**: Prefer partitioning by date/timestamp columns for time-series or event data. Choose the granularity (day, month, year) based on query patterns and data volume.
- **Expiration**: Set partition expiration for temporary, staging, or short-lived data. Retention policies should match business requirements (regulatory, analytical, operational).
- **Partition Pruning**: Ensure queries include partition key filters in `WHERE` clauses to avoid full-table scans.

## Indexing & Clustering
- **Priority**: Index or cluster by columns used in `JOIN` conditions, `WHERE` filters, and `ORDER BY` clauses.
- **Selectivity**: Index high-cardinality columns first. Low-cardinality columns benefit from composite indexes.
- **Limit**: Most engines support 3-4 clustering columns. Order by selectivity (most filtered first).
- **Maintenance**: Monitor index usage and remove unused indexes that add write overhead.

## Query Performance
- **Avoid SELECT \***: Specify only the columns needed, especially in views and subqueries.
- **Filter Early**: Push predicates as close to the data source as possible. Filter before joining, not after.
- **Incremental Builds**: Prefer incremental/append patterns over full table rebuilds for large datasets. Process only new or changed records.
- **Materialized Views**: Use for expensive aggregations that are queried frequently but change infrequently.
- **CTEs vs. Subqueries**: Prefer CTEs for readability, but be aware that some engines materialize CTEs (check engine behavior).

## Cost Management
- **Slot/Compute Reservations**: For analytical databases, understand pricing model (on-demand vs. flat-rate/reserved).
- **Data Scanning**: Structure tables and queries to minimize bytes/rows scanned.
- **Storage Tiers**: Use long-term/archive storage for infrequently accessed data.
- **Query Caching**: Leverage result caching for repeated queries (many engines support this automatically).

## External & Federated Tables
- **Hive Partitioning**: When using external tables over file stores (GCS, S3), ensure the URI pattern matches the partitioning layout (e.g., `gs://bucket/path/key=*/file.parquet`).
- **File Formats**: Prefer columnar formats (Parquet, ORC, Avro) for analytical queries. CSV/JSON for human-readable or streaming ingestion.
- **Schema Drift**: Monitor external table schemas for changes in the source data.
