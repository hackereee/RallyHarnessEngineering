# PLAN-007: Initialize Harness In Real Projects

## Background and Goal

Harness is close to becoming a reusable framework, but the current `project-init` skill is scoped to project environment contracts. This plan turns `project-init` into the top-level real-project onboarding workflow, moves the existing environment contract behavior into a dedicated skill, and adds deterministic entrypoint initialization so real projects can reference the Harness architecture without mixing framework architecture into business architecture.

## Scope

- Rename the existing project environment initialization skill to `project-env-contract`.
- Add a new top-level `project-init` skill for real-project Harness onboarding.
- Add a stable Harness architecture document under `.harness/`.
- Add project entrypoint contract schema/template support.
- Add an idempotent script and CLI command for detecting, creating, and updating agent entrypoint managed blocks.
- Update architecture documentation and tests so the new boundaries are enforced.

## Non-Scope

- Do not parse real project `ARCHITECTURE.md` into a project architecture contract in this plan.
- Do not put project-specific checks into `session-start.py`.
- Do not make entrypoint initialization a workflow task inside target projects.
- Do not write or mutate `workflow-state.json` except through Harness lifecycle tools.

## Implementation Direction

Keep `project-init` as the user-facing onboarding skill. Move the existing environment contract workflow to `project-env-contract`, then make the new `project-init` orchestrate entrypoint detection, `.harness/ARCHITECTURE.md` installation, managed block insertion, entrypoint contract output, and handoff to `project-env-contract`. The entrypoint modification must be deterministic and idempotent through a script; the skill handles semantic choices and conflict review.

## File Boundaries

- Move: `.harness/skills/project-init/SKILL.md` -> `.harness/skills/project-env-contract/SKILL.md`
- Create: `.harness/skills/project-init/SKILL.md`
- Create: `.harness/ARCHITECTURE.md`
- Create: `.harness/schemas/project-entrypoints.schema.json`
- Create: `.harness/templates/project-entrypoints.template.json`
- Create: `.harness/scripts/init-project-entrypoint.py`
- Modify: `.harness/scripts/harness`
- Modify: `.harness/scripts/session-start.py`
- Modify: `harness-design/architecture.md`
- Test: `.harness/tests/test_project_env_contract_skill.py`
- Test: `.harness/tests/test_project_init_skill.py`
- Test: `.harness/tests/test_project_entrypoints_schema.py`
- Test: `.harness/tests/test_init_project_entrypoint.py`
- Test: `.harness/tests/test_harness_cli.py`
- Test: `.harness/tests/test_session_start.py`

## Task Decomposition

This plan uses three delivery tasks: first preserve the existing environment contract workflow under its new name, then add deterministic entrypoint contract/script behavior, then add the new orchestration skill and framework architecture documentation.

## Verification Strategy

Use TDD for each delivery task. Run the focused tests for the changed contract or script after each task, then finish with the full Harness test suite, lint, and workflow-state validation.

## Risks and Open Questions

- Risk: Renaming the skill can break `session-start.py` required assets if tests and documentation are not updated together.
- Risk: Entrypoint insertion can overwrite user rules if it is not constrained to a managed block. The script must only create or replace the marked block.
- Risk: Multiple existing agent entrypoints can drift. The new skill should prefer `AGENTS.md` as canonical and keep other files as short references unless the user chooses otherwise.
- Open questions: None blocking. The user approved the top-level `project-init` plus renamed environment contract skill structure.

## Plan Review Gate

Status: passed
Reviewer: harness-reviewer
Reviewed At: 2026-04-27T23:10:00+08:00

Checks:
- Scope, non-scope, file boundaries, dependencies, acceptance, and verification are reviewable.
- The plan keeps entrypoint detection and managed block updates deterministic.
- The existing project environment contract behavior remains available after the rename.
- The plan does not model testing, review, or initialization gates as independent workflow tasks.

Findings:
- No blocking findings.

## Task Contracts

<a id="task-001-rename-project-env-contract-skill"></a>

### TASK-001: Rename project environment contract skill

Goal: Move the existing project environment contract workflow out of `project-init` and update required assets, tests, and docs to use `project-env-contract`.

Files:
- Create: `.harness/skills/project-env-contract/SKILL.md`
- Modify: `.harness/skills/project-init/SKILL.md`
- Modify: `.harness/scripts/session-start.py`
- Modify: `.harness/tests/test_session_start.py`
- Modify: `.harness/tests/test_project_init_skill.py`
- Create: `.harness/tests/test_project_env_contract_skill.py`
- Modify: `harness-design/architecture.md`

Depends on: []

Acceptance:
- Existing project environment contract requirements are tested under `project-env-contract`.
- `session-start.py` requires `.harness/skills/project-env-contract/SKILL.md`.
- The old environment contract skill content is no longer named `project-init`.
- Architecture docs distinguish the environment contract skill from the new top-level project init skill.

Verification:
- Run: `python3 .harness/tests/test_project_env_contract_skill.py`
- Run: `python3 .harness/tests/test_session_start.py`
- Check: `rg -n "project-env-contract|project-init" .harness/skills .harness/tests .harness/scripts/session-start.py harness-design/architecture.md`

<a id="task-002-add-entrypoint-contract-and-script"></a>

### TASK-002: Add project entrypoint contract and deterministic updater

Goal: Add schema/template and an idempotent script that detects, creates, and updates real-project agent entrypoint documents through a managed Harness block.

Files:
- Create: `.harness/schemas/project-entrypoints.schema.json`
- Create: `.harness/templates/project-entrypoints.template.json`
- Create: `.harness/scripts/init-project-entrypoint.py`
- Modify: `.harness/scripts/harness`
- Modify: `.harness/scripts/session-start.py`
- Test: `.harness/tests/test_project_entrypoints_schema.py`
- Test: `.harness/tests/test_init_project_entrypoint.py`
- Test: `.harness/tests/test_harness_cli.py`
- Test: `.harness/tests/test_session_start.py`

Depends on: [TASK-001]

Acceptance:
- The template validates against the new schema.
- Detect mode reports existing `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, and known tool-specific rule files.
- Missing entrypoints return `NEEDS_ENTRYPOINT`.
- Write mode creates or updates only the `harness-engineering` managed block.
- The script writes `.harness/contracts/project-entrypoints.json` with canonical entry, detected entries, and `.harness/ARCHITECTURE.md` reference.
- The unified CLI exposes `init-entrypoint`.

Verification:
- Run: `python3 .harness/tests/test_project_entrypoints_schema.py`
- Run: `python3 .harness/tests/test_init_project_entrypoint.py`
- Run: `python3 .harness/tests/test_harness_cli.py`
- Run: `python3 .harness/tests/test_session_start.py`

<a id="task-003-add-top-level-project-init-skill-and-docs"></a>

### TASK-003: Add top-level project init skill and Harness architecture doc

Goal: Add the new orchestration skill and stable `.harness/ARCHITECTURE.md`, then update framework architecture documentation to describe how real project entrypoints reference Harness.

Files:
- Create: `.harness/skills/project-init/SKILL.md`
- Create: `.harness/ARCHITECTURE.md`
- Modify: `harness-design/architecture.md`
- Test: `.harness/tests/test_project_init_skill.py`
- Test: `.harness/tests/test_session_start.py`

Depends on: [TASK-001, TASK-002]

Acceptance:
- New `project-init` skill checks for agent entrypoints before writing references.
- It recommends creating `AGENTS.md` when no agent entrypoint exists unless the user chooses another entrypoint.
- It requires `.harness/ARCHITECTURE.md` as the stable Harness framework architecture reference.
- It delegates project environment contract derivation to `project-env-contract`.
- Architecture docs explain that root `ARCHITECTURE.md` is business architecture while `.harness/ARCHITECTURE.md` is framework architecture.

Verification:
- Run: `python3 .harness/tests/test_project_init_skill.py`
- Run: `python3 .harness/tests/test_session_start.py`
- Run: `python3 -m unittest discover -s .harness/tests -p 'test_*.py'`
- Run: `python3 .harness/scripts/lint-harness.py --root .`
- Run: `python3 .harness/scripts/validate-state.py --state work/workflow-state.json --schema .harness/schemas/workflow-state.schema.json`
