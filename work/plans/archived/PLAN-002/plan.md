# PLAN-002: Handoff and Session Start Rules

## Background and Goal

`harness-design/architecture.md` already lists `session-start.md` and `handoff-rules.md` as planned rule documents. The lifecycle implementation has grown enough that this is no longer harmless documentation debt: `session-start.py` writes session audit files, `lifecycle-transaction.py` appends handoff entries, and `plan-writing` creates initial `handoff.md`, but the required semantics are scattered across templates, scripts, and prose.

The goal is to make handoff and session-start behavior a first-class Harness contract: rules live under `.harness/rules/`, active plan handoff shape is checked by lint/tests where deterministic, session-start boundaries are explicit, and architecture/entry documentation points to the same facts.

## Scope

- Add rule documentation for `handoff.md` shape, truth-source boundaries, and lifecycle append records.
- Add rule documentation for `session-start.py` bootstrap, preflight, existing-state read-only behavior, and audit file semantics.
- Align `.harness/templates/handoff.template.md` with the rule contract.
- Add deterministic tests and lint coverage for handoff rule/template invariants where machine-checkable.
- Require the new rule files as Harness assets in `session-start.py`.
- Update architecture and learning/entry docs so they no longer describe these rule documents as planned-only.

## Non-Scope

- Do not change `workflow-state.json` or `tasks.json` schemas unless a machine-expressible field contract is missing.
- Do not model session start, testing, review, or handoff as tasks.
- Do not make `handoff.md` a truth source for workflow or task state.
- Do not automate LLM semantic handoff quality scoring in scripts.
- Do not add the planned `check-env.py` lifecycle tool in this plan.

## Implementation Direction

Keep scripts responsible for deterministic checks and writes. `lint-harness.py` should verify active `handoff.md` has the required sections and header fields, but it should not judge prose quality. `session-start.py` should require the new rule assets and continue to avoid modifying existing state. LLM-owned semantic content remains in handoff/session prose, with `workflow-state.json` and `tasks.json` staying the truth sources.

## File Boundaries

- Create: `.harness/rules/handoff-rules.md`
- Create: `.harness/rules/session-start.md`
- Create: `.harness/tests/test_handoff_rules.py`
- Modify: `.harness/templates/handoff.template.md`
- Modify: `.harness/scripts/lint-harness.py`
- Modify: `.harness/scripts/session-start.py`
- Modify: `.harness/tests/test_lint_harness.py`
- Modify: `.harness/tests/test_session_start.py`
- Modify: `harness-design/architecture.md`
- Modify: `harness-design/handoff.template.md`
- Modify: `AGENTS.md`
- Modify: `README.md`
- Modify: `learning-notes/README.md`

## Task Decomposition

The plan starts with the handoff contract because lifecycle recovery depends on it and because lint can enforce its structural shell. The second task adds session-start rules and asset checks. The final task synchronizes entry documentation and validates the full Harness contract.

## Verification Strategy

Run focused tests after each task. After the plan is complete, run full unittest discovery, `lint-harness.py`, and `validate-state.py`. For pure prose updates, manually check that rule names, truth-source boundaries, and lifecycle terminology are consistent with `workflow-lifecycle.md`.

## Risks and Open Questions

- Risk: Over-validating handoff prose would push semantic judgment into scripts. Keep script checks structural only.
- Risk: `handoff.md` could be mistaken for runtime state. The new rule must explicitly state that it is a recovery summary and never overrides `workflow-state.json` or `tasks.json`.
- Risk: Existing archived handoff files may not match future active lint expectations. Lint should enforce active plan packages only.
- Open questions: None for the initial contract.

## Task Contracts

<a id="task-001-define-handoff-rules"></a>

### TASK-001: Define handoff rules

Goal: Add a rule-backed handoff contract and deterministic structural lint for active plan handoff files.

Files:
- Create: `.harness/rules/handoff-rules.md`
- Create: `.harness/tests/test_handoff_rules.py`
- Modify: `.harness/templates/handoff.template.md`
- Modify: `.harness/scripts/lint-harness.py`
- Modify: `.harness/tests/test_lint_harness.py`
- Modify: `harness-design/handoff.template.md`

Depends on: []

Acceptance:
- `handoff-rules.md` defines truth sources, required sections, required header fields, and lifecycle transaction entry shape.
- `.harness/templates/handoff.template.md` conforms to the handoff rule structure.
- `lint-harness.py` rejects active plan packages whose `handoff.md` is missing required structural sections or required header fields.
- The lint check remains structural and does not judge prose quality or task completion.
- `harness-design/handoff.template.md` points to the canonical `.harness/templates/handoff.template.md` and the new rule document.

Verification:
- Run: `python3 .harness/tests/test_handoff_rules.py`
- Run: `python3 .harness/tests/test_lint_harness.py`
- Check: handoff validation applies to `work/plans/active/<PLAN-ID>/handoff.md` and does not require archived handoff files to change.

<a id="task-002-define-session-start-rules"></a>

### TASK-002: Define session-start rules

Goal: Add explicit session-start lifecycle rules and require them during session preflight.

Files:
- Create: `.harness/rules/session-start.md`
- Modify: `.harness/scripts/session-start.py`
- Modify: `.harness/tests/test_session_start.py`

Depends on: [TASK-001]

Acceptance:
- `session-start.md` states the three allowed startup paths: validate existing state, bootstrap missing state with no active plan, and block missing state with active plan.
- `session-start.md` states that existing `workflow-state.json` must not be modified by session start.
- `session-start.md` defines the session audit file as evidence only, not workflow/task truth source.
- `session-start.py` includes `handoff-rules.md` and `session-start.md` in `REQUIRED_ASSETS`.
- `test_session_start.py` fails when either new rule document is missing from a fixture.

Verification:
- Run: `python3 .harness/tests/test_session_start.py`
- Check: `python3 .harness/scripts/session-start.py --help` still exposes only session-start options and not lifecycle transition behavior.

<a id="task-003-wire-entry-docs-and-full-contract"></a>

### TASK-003: Wire entry docs and full contract

Goal: Synchronize architecture, entry, and learning docs with the new rule documents and prove the full Harness contract still passes.

Files:
- Modify: `harness-design/architecture.md`
- Modify: `AGENTS.md`
- Modify: `README.md`
- Modify: `learning-notes/README.md`
- Test: `.harness/tests/test_session_start.py`
- Test: `.harness/tests/test_lint_harness.py`

Depends on: [TASK-001, TASK-002]

Acceptance:
- `harness-design/architecture.md` lists `session-start.md` and `handoff-rules.md` as implemented rule documents, not planned-only items.
- `AGENTS.md` reading order includes the new rule documents when working on session recovery, handoff, or lifecycle state.
- `README.md` and `learning-notes/README.md` identify session audit and handoff as recovery evidence, not truth sources.
- No documentation contradicts the existing `workflow-state.json` / `tasks.json` truth-source split.
- Full Harness lint and state validation still pass.

Verification:
- Run: `python3 -m unittest discover -s .harness/tests -p 'test_*.py'`
- Run: `python3 .harness/scripts/lint-harness.py --root .`
- Run: `python3 .harness/scripts/validate-state.py --state work/workflow-state.json --schema .harness/schemas/workflow-state.schema.json`
- Check: documentation consistently names `.harness/rules/handoff-rules.md` and `.harness/rules/session-start.md`.
