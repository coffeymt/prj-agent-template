---
name: Entity Resolution
description: 'Patterns for deterministic and probabilistic entity resolution, deduplication, ID generation, and conflict resolution.'
applyTo: "**/*.sql, **/*.sqlx, **/*.py"
---

# Entity Resolution Skill

## Overview
Best practices for resolving, deduplicating, and linking entity records across data sources. Applies to customer identity, address matching, household grouping, and any domain requiring stable, deterministic entity identifiers.

## Core Concepts

### Source Record ID
- **Purpose**: A permanent, immutable identifier for each raw source record.
- **Generation**: Typically a sequential integer (`ROW_NUMBER()`) assigned on first ingestion.
- **Rule**: Once assigned, a source record ID must NEVER be updated or reassigned. It is the audit key linking back to the original input.

### Entity ID (Deterministic Hash)
- **Purpose**: A stable identifier representing a resolved entity (person, address, household, etc.).
- **Generation**: Deterministic hash function (e.g., `FARM_FINGERPRINT`, `MD5`, `SHA256`) applied to a normalized set of key attributes.
- **Properties**: Same inputs always produce the same ID. No randomness. No sequence.
- **Example patterns**:
  - **Address ID**: Hash of (normalized street address + unit/suite)
  - **Household ID**: Hash of (address + last name)
  - **Person ID**: Hash of (address + last name + first name)

### Match Groups
- **Purpose**: Temporary, run-specific groupings produced by matching/linking algorithms.
- **Lifecycle**: Valid only within a single processing run. Not persisted as permanent identifiers.
- **Conflict Resolution**: When multiple records map to the same match group, resolve to a stable entity ID using a deterministic strategy (e.g., `MIN()` of candidate IDs within the group).

## ID Generation Rules

### Immutability
- Source record IDs are permanent. Never regenerate, resequence, or overwrite them.
- Entity IDs are deterministic. The same input attributes always produce the same entity ID.

### Normalization Before Hashing
- Trim whitespace, normalize case (typically UPPER), remove special characters.
- Standardize address components (St → STREET, Ave → AVENUE) before hashing.
- Apply consistent NULL/empty handling — decide whether NULLs are treated as empty strings or excluded.

### Hash Function Selection
- Use a fast, well-distributed hash (`FARM_FINGERPRINT` for BigQuery, `MD5`/`SHA256` for cross-platform).
- Document the exact input formula for each entity ID so it can be reproduced.

## Conflict Resolution Strategies

### MIN() Resolution
- Within a match group, select the minimum entity ID as the canonical representative.
- This ensures deterministic, reproducible results regardless of processing order.

### Recency-Based Resolution
- Prefer the record with the most recent update timestamp.
- Useful when data freshness is more important than stability.

### Confidence Scoring
- Assign match confidence scores and select the highest-confidence link.
- Requires a scoring model — more complex but more accurate for probabilistic matching.

## Safety Checks
- Flag ANY change to ID generation logic as **CRITICAL** — requires explicit approval and downstream impact analysis.
- Changes to hash inputs (adding/removing/reordering fields) will change all generated IDs.
- Verify that temporary/synthetic IDs (e.g., ROW_NUMBER within a processing step) are never persisted to permanent identity tables.
- Any pipeline that writes to identity/dimension tables should have assertions verifying:
  - Uniqueness of primary entity IDs
  - Referential integrity between source records and entity IDs
  - No NULL entity IDs where they are required

## Data Scope
- Keep entity types cleanly separated. A "person" identity table should not contain email-only or device-only records unless the schema explicitly supports it.
- Document which attributes are in scope for each entity type's ID generation.
