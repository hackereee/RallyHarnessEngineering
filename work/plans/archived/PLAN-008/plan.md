# PLAN-008: Add Architecture Impact Gate

## Background and Goal

Harness already separates target project business architecture (`ARCHITECTURE.md`) from Harness framework architecture (`.harness/ARCHITECTURE.md`). The missing lifecycle rule is a required completion-time judgment: after any L0-L3 workflow, the Agent must decide whether the delivered change made either architecture document stale.

The goal is to add an Architecture Impact Gate without modeling "think about architecture" as a fake delivery task.

## Scope

- Add Architecture Impact guidance to planning, review, direct completion, and plan archive flows.
- Make L0/L1 `complete-workflow.py` require a structured architecture impact summary in completion audit records.
- Make L2/L3 `closure.md` require an `Architecture Impact` section before archive.
- Update the plan and closure templates so future workflows capture expected and final architecture impact.
- Update review skill guidance so task review checks actual architecture impact against changed files.
- Add focused regression tests for the new gate.

## Non-Scope

- Do not create a separate task whose only purpose is "decide whether architecture changed".
- Do not parse or score architecture prose automatically.
- Do not update root `ARCHITECTURE.md` or `.harness/ARCHITECTURE.md` for unrelated historical changes.
- Do not change `tasks.json` schema for this gate in this plan.

## Implementation Direction

Treat Architecture Impact as a lifecycle gate:

- Planning records expected architecture impact.
- Task review checks actual architecture impact before a task can pass review.
- L0/L1 completion records final architecture impact in session audit.
- L2/L3 closure records final architecture impact before archive.

The gate remains a semantic LLM judgment, while scripts enforce that the required record exists.

## Architecture Impact

- Expected target project architecture impact: none for this Harness-internal change.
- Expected Harness framework architecture impact: yes. This changes lifecycle rules, templates, scripts, and review skill guidance.
- Expected architecture files to check: `.harness/ARCHITECTURE.md`, `harness-design/architecture.md`, `.harness/rules/workflow-lifecycle.md`, `.harness/rules/archive-rules.md`, `.harness/templates/plan.template.md`, `.harness/templates/closure.template.md`, `.harness/skills/task-review/SKILL.md`.

## File Boundaries

- Modify: `.harness/rules/workflow-lifecycle.md`
- Modify: `.harness/rules/archive-rules.md`
- Modify: `.harness/ARCHITECTURE.md`
- Modify: `harness-design/architecture.md`
- Modify: `.harness/templates/plan.template.md`
- Modify: `.harness/templates/closure.template.md`
- Modify: `.harness/skills/plan-writing/SKILL.md`
- Modify: `.harness/skills/task-review/SKILL.md`
- Modify: `.harness/scripts/complete-workflow.py`
- Modify: `.harness/scripts/archive-plan.py`
- Modify: `.harness/tests/test_complete_workflow.py`
- Modify: `.harness/tests/test_archive_plan.py`
- Modify: `.harness/tests/test_plan_writing_templates.py`

## Task Decomposition

This plan uses one delivery task because the change is a single lifecycle gate that must stay synchronized across rules, templates, scripts, and tests.

## Verification Strategy

Use TDD. First add failing tests that require the architecture impact fields and sections, then update the minimal rules/templates/scripts/skill guidance to pass them. Finish with focused tests plus lint and state validation.

## Risks and Open Questions

- Risk: Over-formalizing the gate could push semantic architecture judgment into scripts. Scripts should only enforce presence; LLM review remains responsible for quality.
- Risk: Treating this as a task would violate Harness task modeling rules. The implementation must explicitly keep it as a gate.
- Open questions: None blocking. The user approved the Architecture Impact Gate direction.

## Plan Review Gate

Status: passed
Reviewer: harness-reviewer
Reviewed At: 2026-04-27T23:55:00+08:00

Checks:
- Scope, non-scope, file boundaries, acceptance, and verification are reviewable.
- The plan models architecture impact as a lifecycle gate, not a standalone task.
- Scripts enforce deterministic presence only; semantic judgment stays with the LLM.
- Testing and review remain workflow gates, not independent tasks.

Findings:
- No blocking findings.

## Task Contracts

<a id="task-001-add-architecture-impact-gate"></a>

### TASK-001: Add architecture impact gate

Goal: Add a lifecycle gate that records expected and final architecture impact for L0-L3 workflows without creating a fake architecture review task.

Files:
- Modify: `.harness/rules/workflow-lifecycle.md`
- Modify: `.harness/rules/archive-rules.md`
- Modify: `.harness/ARCHITECTURE.md`
- Modify: `harness-design/architecture.md`
- Modify: `.harness/templates/plan.template.md`
- Modify: `.harness/templates/closure.template.md`
- Modify: `.harness/skills/plan-writing/SKILL.md`
- Modify: `.harness/skills/task-review/SKILL.md`
- Modify: `.harness/scripts/complete-workflow.py`
- Modify: `.harness/scripts/archive-plan.py`
- Modify: `.harness/tests/test_complete_workflow.py`
- Modify: `.harness/tests/test_archive_plan.py`
- Modify: `.harness/tests/test_plan_writing_templates.py`

Depends on: []

Acceptance:
- Planning templates include an `Architecture Impact` section that distinguishes target project architecture from Harness framework architecture.
- Task review guidance requires checking whether changed files make architecture documents stale.
- L0/L1 completion requires and records architecture impact evidence.
- L2/L3 closure requires an `Architecture Impact` section before archive.
- Rules state that architecture impact is a workflow gate, not a standalone task.

Verification:
- Run: `python3 .harness/tests/test_plan_writing_templates.py`
- Run: `python3 .harness/tests/test_complete_workflow.py`
- Run: `python3 .harness/tests/test_archive_plan.py`
- Run: `python3 .harness/scripts/lint-harness.py --root .`
- Run: `python3 .harness/scripts/validate-state.py --state work/workflow-state.json --schema .harness/schemas/workflow-state.schema.json`
