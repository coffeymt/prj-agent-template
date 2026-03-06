# Stirista AI Agent Team — Strategy & Governance

## Overview

The `prj-agent-template` repository defines an 8-agent AI development team with 8 domain-specific skills, designed for autonomous software engineering on Stirista's data infrastructure. This document covers the team architecture, what it replaces, security guardrails, platform authorization, enterprise distribution, cost governance, and branch protection policy.

---

## 1. Agent Team Architecture

### Agents

| Agent | Model | Role | Writes Code? | Modifies Infra? |
|---|---|---|---|---|
| **Orchestrator** | Claude Opus 4.6 | Decomposes requests into phased plans; delegates to specialists | No | No |
| **Planner** | Gemini 3.1 Pro | Researches codebase, writes PRDs and dependency-aware task graphs | No | No |
| **Architect** | Claude Opus 4.6 | Designs GCP infrastructure blueprints, IAM, networking, service selection | No | No (read-only verify) |
| **Coder** | GPT-5.3-Codex | Writes production code, provisions resources via CLI, fixes build failures | Yes | Yes |
| **Designer** | Gemini 3.1 Pro | UI/UX layouts, Mermaid diagrams, configuration schema design | Yes (design files) | No |
| **Documenter** | Claude Sonnet 4.6 | Updates `assets/` documentation from completed task graphs and git diffs | Yes (docs only) | No |
| **Reviewer** | Claude Sonnet 4.6 | Static code review — correctness, security, patterns. Never writes code | No | No |
| **Auditor** | Gemini 3.1 Pro | Read-only infrastructure verification against blueprints | No | No (read-only) |

### Skills (Domain Knowledge)

| Skill File | Domain | Applies To |
|---|---|---|
| `sql-development.md` | SQL authoring, Dataform/dbt conventions, incremental patterns | `*.sql`, `*.sqlx` |
| `entity-resolution.md` | Deterministic ID generation, deduplication, match groups | `*.sql`, `*.sqlx`, `*.py` |
| `pipeline-orchestration.md` | Cloud Workflows, Pub/Sub, event-driven orchestration | `*.yaml`, `*.yml`, `*.json` |
| `database-optimization.md` | Partitioning, clustering, query cost management | `*.sql`, `*.sqlx`, `*.py` |
| `context-loader.md` | Schema metadata discovery before code generation | `*.sql`, `*.sqlx`, `*.py` |
| `api-contract-validation.md` | Compile-vs-execute variable binding, target mapping | `*.yaml`, `*.sql`, `*.sqlx` |
| `plan-management.md` | Plan directory structure, task graph lifecycle | `plan/**/*.md` |
| `documentation-standards.md` | Writing standards for `assets/` and `docs/` | `assets/**/*.md`, `docs/**/*.md` |

### Execution Flow

```
User Request
    │
    ▼
┌──────────────┐
│ Orchestrator  │  ← Decomposes, delegates, coordinates
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Planner    │  ← PRD + dependency-aware task graph → plan/{task}/
└──────┬───────┘
       │
       ▼ (if infra involved)
┌──────────────┐
│  Architect   │  ← Infrastructure blueprint, IAM design
└──────┬───────┘
       │
       ▼ (parallel phases)
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    Coder     │  │   Designer   │  │    Coder     │  ← Parallel file-scoped tasks
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       ▼                 ▼                 ▼
┌──────────────┐
│   Reviewer   │  ← Static review; CRITICAL → re-Coder loop
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Documenter  │  ← Updates assets/ docs from completed tasks
└──────┬───────┘
       │
       ▼ (if infra changed)
┌──────────────┐
│   Auditor    │  ← Read-only verification of live resources
└──────┬───────┘
       │
       ▼
   Completion Report → User
```

**Key properties:**
- **Plan-first**: Every non-trivial task starts with a Planner-generated PRD and task graph persisted to `plan/{task_name}/`.
- **Phase parallelization**: Tasks with no file overlap execute simultaneously within phases.
- **Mandatory review**: The Reviewer runs after all implementation phases; CRITICAL issues block completion and trigger a fix loop.
- **Mandatory documentation**: The Documenter runs after review passes — no session that produces code skips documentation.
- **Infrastructure audit**: Any session touching cloud resources gets a read-only Auditor verification pass.

---

## 2. What This Replaces (Functional Human Resource Mapping)

| Traditional Role | Agent Replacement | What Changes |
|---|---|---|
| **Technical Project Manager** | Orchestrator + Planner | Task decomposition, phasing, dependency tracking, and delegation are automated. The PM role shifts to reviewing generated plans and approving scope — not creating them. |
| **Solutions Architect** | Architect | Infrastructure blueprints with IAM, networking, and provisioning order are generated from plans. Architects review and approve designs rather than drafting from scratch. |
| **Software Engineer(s)** | Coder (parallelized) | Implementation across SQL, Python, YAML, and config files. Multiple Coder instances run in parallel on non-overlapping files. Engineers shift to reviewing output and handling ambiguous requirements. |
| **UI/UX Designer** | Designer | Dashboard layouts, documentation structure, Mermaid diagrams, config schema design. |
| **Technical Writer** | Documenter | Automated doc updates from task graphs and git diffs. Writers shift to editorial review. |
| **Code Reviewer / Senior Engineer** | Reviewer | Static analysis for correctness, security, hardcoded values, contract violations. Human reviewers focus on business logic judgment calls the Reviewer flags as WARNINGS. |
| **DevOps / SRE** | Auditor | Read-only infrastructure verification against blueprints. SREs focus on remediation, not discovery. |

**Net effect**: Human roles shift from *execution* to *supervision, judgment, and approval*. The agent team handles the mechanical 80% — research, boilerplate, pattern following, validation, documentation — while humans own the 20% that requires business context, ambiguity resolution, and risk tolerance decisions.

---

## 3. Security Guardrails in Agent Declarations

The agent markdown files embed multiple layers of security controls:

### 3.1 Hardcoded Value Prevention (All Agents)

Every implementing agent (Coder, Orchestrator, Architect) enforces:

> *"Do NOT hardcode project IDs, dataset names, connection strings, or environment-specific values in source code. Use configuration, environment variables, or framework references."*
> *"Do NOT hardcode secrets, credentials, or API keys anywhere. Use a secrets manager."*

The Reviewer explicitly checks for these violations in every review and flags them as CRITICAL (blocks merge).

### 3.2 Least Privilege IAM (Architect)

The Architect agent enforces:

> *"Every service account gets only the roles it needs — never `roles/owner` or `roles/editor` in production."*

The Auditor independently verifies IAM bindings and flags overly permissive roles.

### 3.3 Read-Only Boundaries (Reviewer, Auditor)

- **Reviewer**: *"You NEVER write code — you only analyze and report."*
- **Auditor**: *"NEVER run commands that create, modify, or delete resources. Read-only verification only."*

These agents cannot make changes even if prompted to — their tool access is restricted to `[read, search, execute]` (Reviewer) and `[read, search, execute]` (Auditor) without edit permissions.

### 3.4 Critical Change Gating

The Planner flags any change to core identity/key generation logic as:

> *"CRITICAL — requires explicit approval and full downstream impact analysis."*

The Orchestrator cannot bypass this gate — it must surface the flag to the human operator.

### 3.5 Pre-Change Verification

Both Orchestrator and Architect require verifying the current state of target resources before modification:

> *"Before suggesting any schema or infrastructure change, instruct the Architect to verify the current state of the target resource first."*

This prevents blind overwrites and duplicate resource creation.

### 3.6 Tool Restriction per Agent

Each agent's YAML frontmatter limits its available tools:

| Agent | Tools | What's Blocked |
|---|---|---|
| Orchestrator | `read, search, agent` | Cannot edit files or execute commands directly |
| Planner | `read, search, edit, web` | Can write plans but not execute infra commands |
| Architect | `read, search, execute, web` | Can verify (read-only commands) but cannot edit code |
| Coder | `read, search, edit, execute, web` | Full access — the only agent that should make changes |
| Designer | `read, search, edit` | No terminal execution |
| Documenter | `read, search, edit, execute` | Can edit docs and verify via CLI |
| Reviewer | `read, search, execute` | No edit — cannot fix what it finds |
| Auditor | `read, search, execute` | No edit — reports findings only |

### 3.7 Mandatory Review Loop

The Orchestrator enforces a non-optional review cycle:

1. All implementation completes → Reviewer runs
2. CRITICAL issues → Coder fixes → Reviewer re-runs
3. Loop until no CRITICAL issues remain
4. WARNINGS reported to human for judgment

No code reaches the user without passing the Reviewer gate.

---

## 4. Google Platform Authorization via Service Account

### Current Risk: User-Login Authentication

When agents execute `gcloud` commands under a developer's personal login (`gcloud auth login`), they inherit that user's full IAM permissions — often `roles/owner` or broad editor roles. This creates:

- **Privilege escalation**: Agents can access resources beyond their task scope.
- **Audit gap**: All actions appear as the human user in Cloud Audit Logs, making it impossible to distinguish agent actions from human actions.
- **Credential exposure**: Personal OAuth tokens are accessible to any tool with terminal access.

### Recommended: Dedicated Service Account

Provision a purpose-scoped service account for agent-driven operations:

```
agent-pipeline-sa@<project>.iam.gserviceaccount.com
```

**IAM roles** — scoped to exactly what agent tasks require:

| Role | Resource | Purpose |
|---|---|---|
| `roles/bigquery.dataViewer` | Project-level or dataset-level | Read schema metadata, verify tables |
| `roles/bigquery.jobUser` | Project-level | Run queries (compile verification, assertions) |
| `roles/dataform.editor` | Dataform repository | Compile and invoke Dataform |
| `roles/workflows.viewer` | Specific workflows | Describe/verify workflow state |
| `roles/storage.objectViewer` | Specific buckets | Read input data for pipeline context |
| `roles/iam.serviceAccountTokenCreator` | Self | Allow impersonation from developer machines |

**What this service account explicitly does NOT have:**
- `roles/owner`, `roles/editor` — never in any environment
- `roles/iam.admin` — cannot create or modify IAM policies
- `roles/bigquery.dataEditor` — cannot write to production datasets (unless specifically scoped for a deployment step)
- `roles/compute.*` — no VM/compute manipulation
- `roles/resourcemanager.*` — no project-level configuration changes

### Authentication Flow

```
Developer Machine                    GCP
─────────────────                    ───
1. Developer authenticates           (gcloud auth login — personal, used only
   with their own identity            for impersonation grant check)
                                      │
2. Agent activates service account   gcloud auth activate-service-account \
   via key file or workload identity   --key-file=agent-sa-key.json
                                      │
3. All agent gcloud/bq/gsutil        Actions logged as
   commands run as the SA              agent-pipeline-sa@ in Cloud Audit Logs
```

**Preferred: Workload Identity Federation** (no key file required) — if agents run in a CI/CD context (GitHub Actions, Cloud Build), use Workload Identity to bind the GitHub OIDC token to the service account. This eliminates key file management entirely.

### Audit Trail Benefit

With a dedicated SA, Cloud Audit Logs clearly separate:
- **Human actions**: `michael.coffey@stirista.com` — manual operations, approvals
- **Agent actions**: `agent-pipeline-sa@project.iam.gserviceaccount.com` — automated compile, verify, audit

This enables per-principal monitoring, alerting, and cost attribution.

---

## 5. Agent & Skill Distribution to Stirista Employees

### 5.1 The `prj-agent-template` Repository as Distribution Mechanism

The current repo structure **is the distribution mechanism**. The `.github/agents/` and `.github/skills/` directories are portable — they can be:

1. **Cloned directly** into any Stirista project repository
2. **Synced via git submodule** if a single source of truth is preferred
3. **Copied via a bootstrap script** that pulls the latest template into a new project

**Recommended approach**: Maintain `prj-agent-template` as the canonical source. Other repos (e.g., `prj-candlepin`, `prj-lge-prod`) pull agent/skill files from this template via:
- A GitHub Actions workflow that syncs `.github/agents/` and `.github/skills/` on a schedule or on template release
- Or a simple `git subtree` / `git submodule` reference

### 5.2 IDE Dependency — Distribution Varies by Platform

Agent markdown files (`.agent.md`, `.prompt.md`, skills) are currently a **VS Code / GitHub Copilot** convention. Distribution strategy depends on Stirista's IDE standardization decision:

| IDE | Agent Support | Distribution Path |
|---|---|---|
| **VS Code + GitHub Copilot** | Native `.agent.md` and `.github/skills/` support | Repository-based — agents travel with the repo. No extra setup. |
| **Cursor** | Uses `.cursor/rules/` directory with `.mdc` files | Requires a translation layer: convert `.agent.md` → `.mdc` rules. Agents and skills would need reformatting. |
| **JetBrains (AI Assistant)** | Custom prompt profiles, no equivalent of `.agent.md` | Requires manual prompt configuration per developer or a shared settings repo. Most constrained option. |
| **GitHub Copilot (any IDE)** | Copilot Chat with `@workspace` context | Agents/skills work as repo-level instructions if the IDE supports Copilot's agent mode. |

**Recommendation**: Standardize on **VS Code + GitHub Copilot** for teams consuming this agent framework. The `.agent.md` / `.github/skills/` convention is natively supported; no translation needed. Evaluate Cursor as a secondary option since its rules system is conceptually similar but syntactically different.

### 5.3 Limiting LLM Access Under Enterprise Accounts

**GitHub Copilot Enterprise** provides organizational controls:

| Control | Capability |
|---|---|
| **Seat assignment** | Admins assign Copilot licenses to specific users or teams. Unlicensed users have no access. |
| **Policy enforcement** | Organization-level policies control: Copilot Chat availability, agent mode, code completions, suggestions from public code, and which models are available. |
| **Model selection** | Enterprise plans allow restricting which models are available (e.g., allowing only specific models, blocking expensive ones). |
| **Content exclusion** | Define file patterns or repositories that Copilot cannot read or suggest from — protects sensitive IP. |
| **Audit logging** | Copilot usage events are logged per user — who used what, when, and how much. |
| **IP indemnity** | Enterprise plan includes IP indemnity for Copilot-generated code. |

**To limit access:**
1. Assign Copilot Enterprise seats only to approved developers.
2. Configure organization policies to restrict agent mode to specific teams (e.g., the data engineering team).
3. Use content exclusion rules to prevent Copilot from accessing repositories or files containing PII, credentials, or proprietary algorithms.
4. Set model restrictions if certain models have cost or compliance implications.

**For non-Copilot LLM access** (e.g., direct OpenAI/Anthropic/Google API keys):
- Centralize API keys in a secrets manager (GCP Secret Manager, HashiCorp Vault).
- Provision per-team or per-project API keys with spending limits.
- Route all LLM calls through a gateway/proxy that enforces rate limits and logs usage.

---

## 6. Cost Monitoring

### 6.1 GitHub Copilot Enterprise Costs

GitHub Copilot Enterprise pricing is **per-seat, per-month**. Key cost dimensions:

| Dimension | How It Works |
|---|---|
| **Seat cost** | Fixed monthly fee per assigned user. Unassigned users cost nothing. |
| **Premium model usage** | Some models (Claude Opus, GPT-5.3-Codex) consume "premium requests" that may have monthly caps per user. Usage beyond the cap may be throttled or incur overage. |
| **Agent mode interactions** | Each agent invocation (subagent call) counts toward the user's model usage quota. Complex orchestrations with 8+ agent calls per task consume significantly more than simple completions. |

**Monitoring:**
- **GitHub usage dashboard**: Organization admins see per-user Copilot usage (suggestions accepted, chat messages, agent invocations).
- **Billing reports**: Monthly billing breakdown by seat and overage charges.

### 6.2 Can Users Get Different Allotments?

Under standard GitHub Copilot Enterprise licensing:

| Scenario | Supported? | Mechanism |
|---|---|---|
| All users get the same quota | Yes (default) | Standard per-seat licensing |
| Different quotas per user | Not natively | GitHub assigns the same plan features per seat. Differentiation requires assigning users to different plan tiers (Individual vs. Business vs. Enterprise) or using organizational policies to restrict features per team. |
| Power-user vs. light-user tiers | Partially | Assign Copilot seats only to power users. Light users get standard completions without agent mode. |
| Spending caps per user | Not natively per-user | Organization-wide spending limits exist. Per-user caps require a proxy layer or usage monitoring with manual intervention. |

**Practical approach:**
1. Assign **Enterprise seats** (with agent mode) to the core data engineering team.
2. Assign **Business seats** (completions and chat, no agent mode) to other developers.
3. Monitor usage dashboards weekly. If specific users consistently hit premium model caps, audit their workflows for optimization.
4. For direct API access (non-Copilot), use per-project API keys with hard budget ceilings in the cloud provider's billing console.

### 6.3 GCP Cost Attribution for Agent Workloads

Agent-driven `gcloud` and `bq` commands incur GCP costs (BigQuery queries, Cloud Workflow executions, etc.). Track these by:

1. **Labeling**: Apply `created-by: agent-pipeline-sa` labels to all agent-created resources and queries.
2. **Service account filtering**: In GCP Billing Reports, filter by the agent service account to isolate agent-driven spend.
3. **BigQuery audit**: Query `INFORMATION_SCHEMA.JOBS` filtered by `user_email = 'agent-pipeline-sa@...'` to see bytes scanned, slot-seconds, and cost per agent-triggered query.
4. **Budget alerts**: Set GCP budget alerts scoped to the agent service account's project or labeled resources.

---

## 7. Branch Protection & Human-in-the-Loop for Production

### 7.1 Do We Still Need a Human in the Loop?

**Yes.** The agent team is designed with an explicit human approval gate before production merges. The Orchestrator's workflow ends with a *completion report* delivered to the user — it does not auto-merge.

### 7.2 Recommended Branch Protection Rules

Configure these on `main` (and any production-targeted branches) via GitHub repository settings:

| Rule | Setting | Purpose |
|---|---|---|
| **Require pull request** | Enabled | No direct pushes to main — all changes go through a PR. |
| **Required reviewers** | >= 1 human reviewer | At minimum one human must approve before merge. Agent-generated PRs are not self-approving. |
| **Require status checks** | `dataform compile`, lint, test | Automated build must pass. The Coder agent is already trained to run these pre-commit. |
| **Require conversation resolution** | Enabled | All PR review comments must be resolved before merge. |
| **Restrict push access** | Named maintainers only | Only designated team leads can push or merge to main. |
| **Require signed commits** | Recommended | Prevents tampering; especially important when agent-generated commits could be ambiguous. |
| **Dismiss stale reviews** | Enabled | If the branch is updated after approval, the approval resets — prevents approving an earlier version and then sneaking in changes. |
| **Require linear history** | Recommended | Squash-merge or rebase to keep main history clean and auditable. |

### 7.3 Agent Workflow + Branch Protection Interaction

The Orchestrator already includes a **Branch Decision** step:

> *"Use a feature branch when: the change spans multiple files, introduces new functionality, modifies core logic, or could break existing systems."*

The expected flow is:

```
1. Agent creates feature branch   →  feature/add-email-validation
2. Agent implements, reviews,     →  All work on the feature branch
   documents on the branch
3. Agent pushes branch            →  (or developer pushes on agent's behalf)
4. PR created                     →  Automated checks run (compile, lint)
5. Human reviews PR               →  Reviews agent's completion report +
                                      Reviewer's verdict + Auditor's verdict
6. Human approves & merges        →  Only humans with maintainer access
```

**The agent never merges to main.** The human reviews:
- The Reviewer agent's verdict (was the code review clean?)
- The Auditor agent's verdict (is infrastructure correctly provisioned?)
- The Documenter's output (is documentation accurate?)
- Their own judgment on business logic correctness

### 7.4 Preventing Agent Commits to Protected Branches

If agents operate under a dedicated GitHub service account or bot account:

1. **Do not add the bot account to the maintainers list** for protected branches.
2. The bot can push to feature branches and open PRs, but never merge.
3. Configure **CODEOWNERS** to require approval from specific humans for sensitive paths:

```
# .github/CODEOWNERS
definitions/3_0_gold/    @stirista/data-leads
definitions/2_0_silver/  @stirista/data-leads
workflows/               @stirista/platform-leads
.github/agents/          @stirista/ai-governance
.github/skills/          @stirista/ai-governance
```

This ensures changes to production data models, workflows, and the agent definitions themselves require approval from designated humans.

---

## 8. Open Items for Stirista Leadership Discussion

| # | Topic | Decision Needed | Stakeholders |
|---|---|---|---|
| 1 | **IDE Standardization** | Select approved IDE(s) for agent framework compatibility. VS Code is the path of least resistance. | Engineering leads, IT |
| 2 | **Copilot License Tiers** | Determine which teams get Enterprise (agents) vs. Business (completions-only). | Engineering leads, Finance |
| 3 | **Service Account Provisioning** | Create and scope `agent-pipeline-sa` for each project. Decide: key file vs. Workload Identity Federation. | Platform/DevOps, Security |
| 4 | **Template Sync Mechanism** | Choose how other repos consume `prj-agent-template`: submodule, subtree, or GitHub Actions sync. | Engineering leads |
| 5 | **Per-User Cost Caps** | Decide whether to implement a usage proxy for per-user LLM spending limits or rely on seat-level controls. | Finance, Engineering leads |
| 6 | **Agent Governance Ownership** | Assign a team/person responsible for maintaining agent definitions and skills — the `@stirista/ai-governance` CODEOWNER. | Engineering leads, Management |
| 7 | **Content Exclusion Policies** | Define which repos/files Copilot should be excluded from reading (PII datasets, proprietary matching algorithms, credentials). | Security, Legal |
| 8 | **Merge Authority** | Formalize who has maintainer access on production branches per repository. | Engineering leads |
