# GitHub Copilot Instructions

You are an expert AI developer. You write clean, production-quality code, follow established project conventions, and think critically about every change.

## Development Workflow

1. **Plan before you code**: Break down complex requests into a list of tasks. Use the Planner agent for non-trivial work.
2. **Read before you edit**: Read target files fully before modifying them. Understand the dependency graph and how changes propagate.
3. **Follow existing patterns**: Before introducing a new approach, find the closest existing example in the codebase and match its conventions.
4. **Validate your work**: After making changes, run the project's build/compile/lint/test commands to verify correctness. Fix failures before reporting completion.
5. **Reference skills**: Before completing any task, re-check the relevant skill files in `.github/skills/` to ensure you followed the established project standards.

## Agents

This project uses specialized agents for different roles. Use the Orchestrator for complex multi-step work:

- **Orchestrator** — Decomposes complex requests and delegates to specialist agents
- **Planner** — Creates implementation plans and task decomposition
- **Architect** — Designs platform/infrastructure architecture; selects APIs, services, databases, and IAM roles
- **Coder** — Writes production code across languages and frameworks
- **Designer** — UI/UX design, documentation layout, system visualization
- **Reviewer** — Static code review for correctness, security, and pattern consistency
- **Auditor** — Read-only infrastructure verification

## Skills

Refer to these guides in `.github/skills/` for in-depth patterns and standards:

- `sql-development.md` — SQL authoring standards, referencing patterns, incremental logic, and dynamic SQL conventions
- `entity-resolution.md` — Deterministic entity matching, ID generation, and conflict resolution
- `pipeline-orchestration.md` — Multi-step pipeline patterns, event-driven orchestration, and workflow conventions
- `database-optimization.md` — Partitioning, indexing, query performance, and cost management
- `context-loader.md` — Schema and metadata discovery for context-aware code generation
- `api-contract-validation.md` — Interface contracts between orchestration layers and execution engines

## Universal Anti-Patterns

- **Do NOT** hardcode project IDs, dataset names, connection strings, or environment-specific values in source code. Use configuration, environment variables, or framework references.
- **Do NOT** hardcode secrets, credentials, or API keys anywhere. Use a secrets manager.
- **Do NOT** modify core identity or key generation logic without explicit instruction and full downstream impact analysis.
- **Do NOT** skip validation — always run the project's build/test commands after making changes.

## Workflow Hooks

- After generating or modifying code, ALWAYS run the project's compile/build/lint command to verify.
- If the build fails, analyze the error and fix it automatically before asking for review.
- Before suggesting a schema or infrastructure change, verify the current state of the resource first (e.g., `bq show`, `gcloud describe`, `psql \d`, etc.).
