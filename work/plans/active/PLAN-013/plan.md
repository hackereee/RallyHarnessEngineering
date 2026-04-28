# PLAN-013: Standardize .harness Human-Facing Language to English

## Background and Goal

Source backlog: BL-007 (`chat:2026-04-29-harness-language-standardization`).

The `.harness` framework currently mixes English and Chinese across architecture documentation, rules, templates, schemas, script comments, diagnostics, and test expectations. This plan standardizes human-facing Harness framework assets to English while preserving the established Harness lifecycle, schema, script gateway, and workflow gate behavior.

## Scope

- Normalize `.harness` architecture, rule, skill, template, and schema prose to English.
- Normalize `.harness/scripts` module docstrings, comments, CLI help, diagnostics, and emitted user-facing messages to English.
- Update related `.harness/tests` expectations and add a regression guard for language standardization.
- Preserve Harness runtime boundaries: scripts remain deterministic gateways, `workflow-state.json` writes remain behind `state-write.py`, and `tasks.json` writes remain behind `update-task.py` or `materialize-tasks.py`.

## Non-Scope

- Do not translate root `AGENTS.md`, root `README.md`, root `ARCHITECTURE.md`, or any non-`.harness` project document.
- Do not change Harness task lifecycle semantics, phase transition rules, schema shape, task review thresholds, or script side effects.
- Do not model testing, review, backlog consumption, commit, handoff, or architecture impact as standalone delivery tasks.
- Do not hand-edit generated `tasks.json` runtime state.

## Implementation Direction

Treat this as an L3 decomposed Harness framework cleanup because it touches the contract layer, script layer, and regression suite. Work in dependency order: first normalize static framework contracts and documentation, then normalize scripts and paired expectations, then add a repository-level language guard and run the broader Harness regression set. Any remaining non-English text inside `.harness` must be treated as a blocking finding unless it is a deliberate compatibility fixture with explicit test coverage and a documented reason.

## File Boundaries

- Modify: `.harness/ARCHITECTURE.md`
- Modify: `.harness/rules/`
- Modify: `.harness/skills/`
- Modify: `.harness/templates/`
- Modify: `.harness/schemas/`
- Modify: `.harness/scripts/`
- Modify: `.harness/tests/`
- Create: `.harness/tests/test_language_standardization.py`

## Task Decomposition

The work is split by artifact ownership. Static contracts and prose come first because scripts and tests refer to the same terms. Script diagnostics and behavior expectations come second. A final regression guard and full verification pass closes the plan.

## Verification Strategy

Each task must run the focused tests for its touched artifact family. The final task must run the broader Harness regression set and a language scan over `.harness`. Verification evidence belongs in task `verification`, session audit, and review summaries; it must not be represented as a separate task.

## Architecture Impact

- Expected target project architecture impact: root `ARCHITECTURE.md` should remain unchanged because this work only changes the Harness framework assets under `.harness`.
- Expected Harness framework architecture impact: `.harness/ARCHITECTURE.md`, rules, templates, schemas, scripts, skills, and tests are expected to change only for English normalization and regression coverage. Lifecycle semantics and directory responsibilities must remain unchanged.
- This is a workflow gate record, not a standalone task. Any architecture documentation edit is part of the affected deliverable task.

## Risks and Open Questions

- Risk: A naive text replacement can alter script behavior, especially validation heuristics and test fixtures. The implementation must distinguish user-facing diagnostics from intentional compatibility data.
- Risk: Some tests assert exact diagnostic text. Script and test changes must land in the same task boundary when needed.
- Open questions: None blocking. The backlog already defines the target language and scope.

## Plan Review Gate

Status: passed
Reviewer: harness-reviewer
Reviewed At: 2026-04-29T07:04:17+08:00

Checks:
- Scope, non-scope, file boundaries, dependencies, acceptance, and verification are reviewable.
- Task contracts are parseable and contain no testing-only or review-only tasks.
- Architecture Impact is recorded as a gate, not a standalone task.
- Testing and review remain workflow gates, not standalone tasks.
- The plan does not require direct writes to `workflow-state.json` or hand-authored `tasks.json`.

Findings:
- No blocking findings.

## Task Contracts

<a id="task-001-normalize-framework-contract-prose"></a>

### TASK-001: Normalize framework contract prose

Goal: Translate and standardize static Harness framework documentation, rules, skills, templates, and schema prose to English.

Files:
- Modify: `.harness/ARCHITECTURE.md`
- Modify: `.harness/rules/`
- Modify: `.harness/skills/`
- Modify: `.harness/templates/`
- Modify: `.harness/schemas/`
- Test: `.harness/tests/test_plan_writing_templates.py`
- Test: `.harness/tests/test_tasks_schema.py`
- Test: `.harness/tests/test_workflow_state_schema.py`

Depends on: []

Acceptance:
- Static `.harness` framework prose is English across architecture, rules, skills, templates, and schemas.
- Schema descriptions, `$comment` values, template text, and skill instructions preserve their existing Harness semantics.
- No task, testing, review, backlog, handoff, commit, or archive lifecycle boundary is weakened or moved out of its established layer.

Verification:
- Run: `python3 .harness/tests/test_plan_writing_templates.py`
- Run: `python3 .harness/tests/test_tasks_schema.py`
- Run: `python3 .harness/tests/test_workflow_state_schema.py`
- Check: `rg -n "\p{Han}" .harness/ARCHITECTURE.md .harness/rules .harness/skills .harness/templates .harness/schemas` reports no remaining human-facing Chinese prose.

<a id="task-002-normalize-script-diagnostics"></a>

### TASK-002: Normalize script diagnostics

Goal: Translate and standardize `.harness/scripts` comments, docstrings, CLI help, errors, warnings, and emitted user-facing messages to English.

Files:
- Modify: `.harness/scripts/`
- Modify: `.harness/tests/`
- Test: `.harness/tests/test_validate_state.py`
- Test: `.harness/tests/test_lint_harness.py`
- Test: `.harness/tests/test_state_write.py`
- Test: `.harness/tests/test_harness_cli.py`

Depends on: [TASK-001]

Acceptance:
- Script comments and docstrings are English.
- CLI help, warnings, errors, success messages, and JSON-facing diagnostic strings are English.
- Existing script gateway responsibilities and side effects are unchanged.
- Tests that assert exact diagnostics are updated to the new English wording without relaxing the behavioral assertions.

Verification:
- Run: `python3 .harness/tests/test_validate_state.py`
- Run: `python3 .harness/tests/test_lint_harness.py`
- Run: `python3 .harness/tests/test_state_write.py`
- Run: `python3 .harness/tests/test_harness_cli.py`
- Check: `rg -n "\p{Han}" .harness/scripts` reports no remaining human-facing Chinese script text.

<a id="task-003-add-language-regression-guard"></a>

### TASK-003: Add language regression guard

Goal: Add a regression test and final validation pass that keeps `.harness` human-facing assets standardized in English.

Files:
- Create: `.harness/tests/test_language_standardization.py`
- Modify: `.harness/tests/`
- Test: `.harness/tests/test_language_standardization.py`
- Test: `.harness/tests/test_materialize_tasks.py`
- Test: `.harness/tests/test_lifecycle_transaction.py`
- Test: `.harness/tests/test_backlog_consume.py`

Depends on: [TASK-001, TASK-002]

Acceptance:
- A dedicated language-standardization test fails on accidental Chinese human-facing text under `.harness`.
- Existing Harness regression tests pass with English diagnostics and expectations.
- `rg -n "\p{Han}" .harness` has no unexplained matches after the plan completes.

Verification:
- Run: `python3 .harness/tests/test_language_standardization.py`
- Run: `python3 .harness/tests/test_materialize_tasks.py`
- Run: `python3 .harness/tests/test_lifecycle_transaction.py`
- Run: `python3 .harness/tests/test_backlog_consume.py`
- Run: `python3 .harness/tests/test_lint_harness.py`
- Check: `rg -n "\p{Han}" .harness` reports no remaining unexplained matches.
