# GitHub Copilot Instructions for prj-candlepin

You are an expert AI developer working on `prj-candlepin`, an Identity Management (IDM) System built with **Dataform** on **BigQuery**.

## 🏗️ Project Architecture & Data Flow

This project follows a strict Medallion Architecture (Iron → Bronze → Silver → Gold → Platinum → Diamond) to manage terrestrial name and address data.

- **0_Iron**: Raw data declarations and imports.
- **1_Bronze**: Cleaned, normalized inputs.
- **2_Silver**: Core identity resolution, Melissa matching, and historization.
- **3_Gold**: Business logic, dimensions, and fact tables.
- **4_Platinum**: Advanced analytics, aggregates, and machine learning features.
- **5_Diamond**: Executive dashboards, table objects for application consumption, etc.

### 🔑 Core Identity Concepts (CRITICAL)
Refer to `assets/IDENTITY_MANAGEMENT_README.md` for the source of truth.
- **SCID (Source Customer ID)**: Permanent, immutable, sequential ID for a source record. Assigned in `x_initial_customer`.
- **AID (Address ID)**: Deterministic hash of validated delivery point (DPV + Address2).
- **HID (Household ID)**: Deterministic hash of DPV + Address2 + Last Name.
- **CID (Customer ID)**: Deterministic hash of DPV + Address2 + Full Name.
- **Match Groups**: Run-specific IDs (`h_group_num`) from Melissa Data, used for conflict resolution via `MIN()` logic.

**Rule**: Never modify the logic for SCID/AID/HID/CID generation without explicit instruction and deep understanding of `s_customer_matched.sqlx`.

## 🛠️ Dataform & SQL Patterns

- **References**: Always use `${ref("table_name")}` for dependencies. Never hardcode table names.
- **Incremental Logic**: Use `${when(incremental(), ...)}` for efficient updates.
  - Example: `definitions/1_0_bronze/aa_consumer.sqlx`
- **Dynamic SQL**: Complex logic often uses `pre_operations` to generate dynamic SQL strings (e.g., `definitions/2_2_silver/s_customer.sqlx`).
- **Configuration**: `workflow_settings.yaml` defines project defaults (schema, location).

## 🔄 Development Workflow

1.  **Taskmaster Mindset**: Break down complex requests into a list of tasks. Plan before you code.
2.  **Safety First**: Read `.sqlx` files fully before editing to understand the dependency graph.
3.  **Testing**: Verify logic by checking `assertions/` (e.g., uniqueness, referential integrity).
4.  **Reference Docs Before Done**: Before completing any task, re-check this document **and** the Skills docs it links to (`.github/skills/`) to ensure you followed the established project patterns.

## 🧭 1PD vs 3PD Consistency Rule

Any development work for **1PD tasks** in `prj-candlepin-lg-1pd/` must **mimic the established Dataform project structure and SQLX patterns** already implemented in the **3PD environment** `prj-candlepin/` (configs, naming, refs, incremental patterns, and general formatting). When in doubt, find the closest existing 3PD example and match its approach.

## 📂 Key Files & Directories

- `assets/IDENTITY_MANAGEMENT_README.md`: **READ THIS FIRST**. The bible for identity logic.
- `assets/ORCHESTRATOR_WORKFLOW_RUNBOOK.md`: The bible for multi-tenant data orchestration and VM integration.
- `assets/CLIENT_ONBOARDING_RUNBOOK.md`: Guide for onboarding new tenants/clients (IAM, Buckets, Scheduler).
- `assets/CONSUMER_PIPELINE_README.md`: Detailed architecture and data flow for the consumer pipeline.
- `definitions/2_0_silver/qualified_customers.sqlx`: Union of all bronze source inputs; entry point for new data.
- `definitions/2_2_silver/s_customer.sqlx`: Dynamic source unification and valid input prep.
- `definitions/2_3_silver/s_customer_matched.sqlx`: Core matching and ID generation logic.
- `definitions/2_3_silver/op_update_customer_processed_flags.sqlx`: Closes the ingestion loop by marking sources as processed.
- `definitions/2_3_silver/s_fl_customer_history.sqlx`: Historical append-only table (7-year retention).
- `definitions/3_0_gold/d_customer.sqlx`: Final "Golden Record" dimension aggregated by Individual (CID).
- `workflow_settings.yaml`: Dataform configuration.

## � Documentation & Skills

Refer to these specific guides for in-depth patterns and rules:

- **Skills** (in `.github/skills/`):
  - `dataform-sqlx.md`: SQLX authoring standards, config rules, and dynamic SQL patterns.
  - `identity-resolution.md`: Deep dive into SCID/AID/HID/CID logic and conflict resolution.
  - `gcp-workflow.md`: Orchestration details for Dataform <-> Workflows <-> GCS.
  - `bigquery-optimization.md`: Partitioning, clustering, and performance best practices.

## 🚫 Anti-Patterns

- **Do NOT** hardcode project IDs or dataset names in SQLX files.
- **Do NOT** suggest changing the `SCID` generation logic (it must remain permanent and sequential).

## Workflow Hooks

- After generating or modifying any `.sqlx` file, ALWAYS run the terminal command `dataform compile`.
- If the compilation fails, analyze the error and fix the SQL automatically before asking for review.
- Before suggesting a BigQuery schema change, run `bq show --schema your_project:dataset.table` to verify existing fields.
