---
name: Plan Management
description: 'Conventions for structuring, persisting, and tracking implementation plans across planner, orchestrator, and implementation agents.'
applyTo: "plan/**/*.md"
---

# Plan Management Skill

## Overview
Use this skill to define a consistent planning contract across Planner, Orchestrator, and implementation agents (Coder, Architect, Designer).

## Plan Directory Convention
- Store each implementation plan under `plan/{task_name}/`.
- `{task_name}` must be a kebab-case slug derived from the feature/task name.
- Example: `plan/add-auth-module/`.
- Each plan directory contains exactly two files:
  - `prd.md`
  - `tasks.md`
- Do not add extra files unless explicitly approved.

## `prd.md` Required Structure
Use these sections in order:
1. **Problem Statement**
2. **Goals**
3. **Capability Tree**
4. **Repository Structure**
5. **Dependency Chain**
6. **Development Phases**
7. **Out of Scope**
8. **Constraints**
9. **Open Questions**

Guidance:
- Keep statements concrete and implementation-oriented.
- Map capabilities to code areas and owning agents.
- Make dependencies explicit (what must exist before what).

## `tasks.md` Required Structure
- Use GitHub-flavored markdown checkboxes:
  - Pending: `- [ ]`
  - Complete: `- [x]`
- Organize tasks into phases that reflect execution order and dependencies.
- Each task line must include:
  - Checkbox
  - Task ID
  - Description
  - Assigned agent
  - Target files

Required task format:
```md
- [ ] **Task 1.1** — Create user model -> Coder | Files: `src/models/user.ts`
```

Task authoring rules:
- Use stable IDs (`Task 1.1`, `Task 1.2`, `Task 2.1`, ...).
- Keep descriptions action-oriented and testable.
- List all touched files; use `TBD` only when genuinely unknown.
- Include dependency notes at phase boundaries when needed.

## Task Lifecycle
1. Planner creates `plan/{task_name}/prd.md` and `plan/{task_name}/tasks.md`.
2. Orchestrator reads `tasks.md` to determine phase-by-phase execution.
3. Implementation agents execute assigned tasks and update completion state by replacing `- [ ]` with `- [x]`.
4. Orchestrator tracks progress by reading remaining unchecked tasks.

## Execution Expectations
- Complete all tasks in a phase before starting dependent phases unless `tasks.md` explicitly allows overlap.
- Update task checkboxes immediately after completion to keep orchestration state accurate.
- If scope changes, Planner updates both `prd.md` and `tasks.md` before new implementation starts.

## Guardrails
- Follow existing project patterns for structure, naming, and dependencies.
- Do not hardcode project IDs, dataset names, connection strings, or environment-specific values.
- Keep plans implementation-facing: explicit dependencies, explicit owners, explicit target files.
