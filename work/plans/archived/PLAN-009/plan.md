# PLAN-009: Project Init Entrypoint Workflow Integration

## Review Findings Before Plan

No direction-level reversal is needed: `project-init` should connect target project entrypoints to Harness workflow semantics without copying this repository's full `AGENTS.md` into the target project.

Corrections and missing points found during draft review:

- The review scope must cover all detected agent entrypoints, not only canonical `AGENTS.md`. Tool-specific files such as `CLAUDE.md`, `GEMINI.md`, Copilot instructions, and editor rule files can contain workflow instructions that conflict with Harness lifecycle.
- The managed block needs a deterministic version or profile marker. Current `project-entrypoints.json` records only `harnessBlock = present|absent|not-applicable`, so an old block can be reported as present even when it lacks the new workflow integration rules.
- The semantic workflow integration review should remain an LLM responsibility. Do not make `init-project-entrypoint.py` parse freeform entrypoint prose for conflicts.
- Deterministic block version, block presence, canonical entry, and architecture refs can be schema/script/test governed.
- The old `harness-design/architecture.md` path is already removed. New architecture updates should target `.harness/ARCHITECTURE.md` and relevant `.harness/rules/` documents only.

## Background and Goal

`project-init` currently initializes target project agent entrypoints by detecting known entry files, installing a Harness managed block, ensuring root `ARCHITECTURE.md`, and writing `.harness/contracts/project-entrypoints.json`.

The missing behavior is workflow integration: target projects often already encode how agents should plan, implement, test, review, commit, and hand off work. `project-init` should require the Agent to inspect those existing workflow rules, map them onto Harness phases and gates, and surface conflicts before writing or recommending changes.

The goal is to strengthen `project-init` so target `AGENTS.md` becomes a concise workflow entrypoint:

- it keeps target project rules valid where they do not conflict with Harness;
- it maps project workflow semantics to Harness lifecycle instead of creating a second state machine;
- it identifies truth sources and write gateways;
- it keeps environment commands in project contracts;
- it avoids copying this repository's full `AGENTS.md` into target projects.

## Scope

- Add a `Workflow Integration Review` section to `.harness/skills/project-init/SKILL.md`.
- Define what target entrypoint content belongs in the managed block versus outside the managed block.
- Update `init-project-entrypoint.py` managed block content to include workflow mapping, conflict priority, truth sources, gate boundaries, and write gateway rules.
- Add deterministic managed block versioning so stale blocks can be distinguished from current blocks.
- Extend `.harness/schemas/project-entrypoints.schema.json` and `.harness/templates/project-entrypoints.template.json` only for deterministic fields such as managed block version or profile.
- Add or revise tests for skill guidance, managed block content, schema/template validation, and script idempotency.
- Update `.harness/ARCHITECTURE.md` to document the target entrypoint integration boundary.

## Non-Scope

- Do not copy this repository's root `AGENTS.md` into target projects.
- Do not rewrite target entrypoint prose outside the `harness-engineering` managed block.
- Do not make scripts judge semantic conflicts in freeform entrypoint text.
- Do not add project-specific build, test, runtime, service, or secret requirements to `session-start.py`.
- Do not turn testing, review, architecture impact, commit, or handoff into standalone tasks.

## Implementation Direction

Keep a strict split:

- `project-init` skill performs semantic review: read detected entrypoints, identify workflow rules, map them to Harness lifecycle, and report conflicts or required manual edits.
- `init-project-entrypoint.py` performs deterministic updates: create or replace only the managed block, ensure root `ARCHITECTURE.md`, and write the entrypoint contract.
- `project-entrypoints.schema.json` records deterministic entrypoint facts, including block version if added.
- `.harness/contracts/project-contracts.json` remains the destination for project environment commands and checks.

The target managed block should be short enough to live in every target `AGENTS.md`, but strong enough to prevent workflow drift.

## Target AGENTS.md Managed Block Content

The managed block should include these entrypoint-level rules:

1. Read order:
   - current agent entrypoint;
   - root `ARCHITECTURE.md`;
   - `.harness/ARCHITECTURE.md`;
   - `.harness/rules/workflow-lifecycle.md`;
   - scenario rules: `session-start.md`, `handoff-rules.md`, `archive-rules.md`, `backlog-rules.md`;
   - `.harness/contracts/`.
2. Conflict priority:
   - target project rules remain valid when compatible;
   - workflow, task, testing, review, state, commit, handoff, backlog, or archive conflicts must be mapped to Harness lifecycle;
   - the Agent must report conflicts before changing user prose.
3. Workflow mapping:
   - startup and resume rules map to `session-start`;
   - planning maps to `planning`;
   - development maps to `implementing`;
   - tests map to `testing` gate;
   - reviews map to `reviewing` gate;
   - task completion commits map to `commit-task.py`;
   - L0/L1 completion maps to `complete-workflow.py`;
   - L2/L3 archive maps to `archive-plan.py`;
   - incoming work maps to `backlog-intake.py`.
4. Truth sources:
   - workflow runtime: `work/workflow-state.json`;
   - task runtime: `work/plans/active/<PLAN-ID>/tasks.json`;
   - planning contract: `work/plans/active/<PLAN-ID>/plan.md`;
   - recovery summary: `handoff.md` only summarizes state;
   - project environment contract: `.harness/contracts/project-contracts.json`;
   - project entrypoint contract: `.harness/contracts/project-entrypoints.json`.
5. Write gateways:
   - `workflow-state.json` only through `state-write.py` or lifecycle tools that call it;
   - `tasks.json` only through `materialize-tasks.py` for initialization and `update-task.py` for updates;
   - phase transitions through `lifecycle-transaction.py` when available;
   - backlog writes through `backlog-intake.py`.
6. Task modeling:
   - task means deliverable unit;
   - testing, review, architecture impact, commit, and handoff are gates or audit actions, not tasks;
   - L0/L1 do not create plans;
   - L2/L3 use one active plan and one active task during implementing/testing/reviewing.

## File Boundaries

- Modify: `.harness/skills/project-init/SKILL.md`
- Modify: `.harness/scripts/init-project-entrypoint.py`
- Modify: `.harness/schemas/project-entrypoints.schema.json`
- Modify: `.harness/templates/project-entrypoints.template.json`
- Modify: `.harness/ARCHITECTURE.md`
- Test: `.harness/tests/test_project_init_skill.py`
- Test: `.harness/tests/test_init_project_entrypoint.py`
- Test: `.harness/tests/test_project_entrypoints_schema.py`

## Task Decomposition

The implementation is split into three delivery tasks:

- `TASK-001` strengthens the semantic `project-init` workflow integration guidance.
- `TASK-002` updates deterministic managed block generation and entrypoint contract metadata.
- `TASK-003` documents the integration boundary in Harness architecture.

Testing, review, architecture impact judgment, lifecycle handoff, and commit gates are workflow gates or audit actions, not standalone tasks.

## Verification Strategy

Run the task-specific tests listed in each task contract. After lifecycle or architecture documentation changes, run `lint-harness.py` and `validate-state.py` against the active workflow state.

## Architecture Impact

- Expected target project architecture impact: target projects will get a stronger generated `AGENTS.md` managed block, but their root `ARCHITECTURE.md` should remain business architecture and should not receive Harness framework prose.
- Expected Harness framework architecture impact: yes. `.harness/ARCHITECTURE.md` should document that project entrypoint integration maps target workflow rules to Harness lifecycle and does not create a parallel state machine.
- This is a lifecycle/entrypoint integration gate, not a standalone task.

## Risks and Open Questions

- Risk: Overloading the managed block can make target `AGENTS.md` noisy. Keep it as entrypoint rules and link to deeper Harness docs for details.
- Risk: Recording semantic conflict review in schema would create false determinism. Keep semantic findings in Agent output or session audit, not as contract truth.
- Risk: Adding managed block versioning requires script, schema, template, and tests to change together.
- Open question: The exact version field name should be chosen during implementation. Recommended shape is either top-level `managedBlockVersion` or per-entry `harnessBlockVersion`; per-entry is more precise if non-canonical entrypoints are later updated.

## Plan Review Gate

Status: passed
Reviewer: user-approved-draft-review
Reviewed At: 2026-04-28T08:34:58+08:00

Checks:
- Scope and non-scope preserve the LLM/script boundary.
- Task contracts are parseable and do not model testing/review as tasks.
- Schema changes are limited to deterministic block metadata.
- Managed block wording is concise enough for target `AGENTS.md`.
- `.harness/ARCHITECTURE.md` is updated, not the removed legacy architecture path.

Findings:
- No blocking findings.

## Task Contracts

<a id="task-001-strengthen-project-init-skill"></a>

### TASK-001: Strengthen project-init workflow integration guidance

Goal: Make `project-init` require semantic workflow integration review across all detected target entrypoints before recommending or writing managed block changes.

Files:
- Modify: `.harness/skills/project-init/SKILL.md`
- Test: `.harness/tests/test_project_init_skill.py`

Depends on: []

Acceptance:
- The skill requires reading all detected entrypoints before workflow conclusions.
- The skill defines workflow mapping from target project rules to Harness phases and gates.
- The skill requires conflict reporting before modifying user-owned prose.
- The skill states that semantic conflict judgment belongs to the LLM, not `init-project-entrypoint.py`.
- The skill keeps project environment commands delegated to `project-env-contract`.

Verification:
- Run: `python3 .harness/tests/test_project_init_skill.py`
- Check: `rg -n "Workflow Integration Review|testing.*gate|reviewing|state-write.py|project-env-contract" .harness/skills/project-init/SKILL.md .harness/tests/test_project_init_skill.py`

<a id="task-002-version-and-update-managed-block"></a>

### TASK-002: Version and update the managed block

Goal: Update deterministic entrypoint management so generated target managed blocks carry the workflow integration rules and can be identified as current or stale.

Files:
- Modify: `.harness/scripts/init-project-entrypoint.py`
- Modify: `.harness/schemas/project-entrypoints.schema.json`
- Modify: `.harness/templates/project-entrypoints.template.json`
- Test: `.harness/tests/test_init_project_entrypoint.py`
- Test: `.harness/tests/test_project_entrypoints_schema.py`

Depends on: [TASK-001]

Acceptance:
- Managed block content includes read order, conflict priority, workflow mapping, truth sources, write gateways, and task modeling boundaries.
- The script still modifies only the managed block.
- The script remains idempotent when run twice.
- The contract records deterministic block version/profile metadata.
- The schema and template validate the new metadata.
- Existing entrypoint prose outside the markers remains unchanged.

Verification:
- Run: `python3 .harness/tests/test_init_project_entrypoint.py`
- Run: `python3 .harness/tests/test_project_entrypoints_schema.py`
- Check: a fixture with an old block is replaced exactly once and user prose before/after the block is preserved.

<a id="task-003-document-entrypoint-integration-boundary"></a>

### TASK-003: Document the entrypoint integration boundary

Goal: Update Harness architecture documentation so future maintainers know what belongs in target `AGENTS.md` managed blocks and what remains in Harness docs, scripts, schemas, and contracts.

Files:
- Modify: `.harness/ARCHITECTURE.md`
- Test: `.harness/tests/test_project_init_skill.py`

Depends on: [TASK-001, TASK-002]

Acceptance:
- `.harness/ARCHITECTURE.md` documents target agent entrypoint integration as a workflow mapping layer.
- It states that root `ARCHITECTURE.md` remains target project business architecture.
- It states that `.harness/ARCHITECTURE.md` remains Harness framework architecture.
- It states that `project-entrypoints.json` is deterministic entrypoint metadata, not a semantic conflict report.
- It avoids references to removed `harness-design/architecture.md`.

Verification:
- Run: `python3 .harness/tests/test_project_init_skill.py`
- Run: `python3 .harness/scripts/lint-harness.py --root .`
- Run: `python3 .harness/scripts/validate-state.py --state work/workflow-state.json --schema .harness/schemas/workflow-state.schema.json`
