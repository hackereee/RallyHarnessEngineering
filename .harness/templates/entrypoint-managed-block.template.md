<!-- harness-engineering:start -->
## Harness Engineering

This repository uses Harness Engineering for agent workflow control.

Managed block version: `harness-entrypoint-block-v2`

Read order:
1. This agent entry document.
2. Project business architecture: `ARCHITECTURE.md`.
3. Harness framework architecture: `.harness/ARCHITECTURE.md`.
4. Harness task level rules: `.harness/rules/task-level.md`.
5. Harness lifecycle rules: `.harness/rules/workflow-lifecycle.md`.
6. Scenario rules: `.harness/rules/session-start.md`, `.harness/rules/handoff-rules.md`, `.harness/rules/archive-rules.md`, `.harness/rules/backlog-rules.md`, `.harness/rules/llm-script-boundary.md`, and `.harness/rules/workflow-gates.md`.
7. Harness project contracts: `.harness/contracts/`.

Conflict priority:
- Target project rules remain valid when compatible with Harness lifecycle.
- Workflow, task, testing, review, state, commit, handoff, backlog, or archive conflicts map to Harness lifecycle.
- Report conflicts before changing user-owned prose outside this managed block.

Workflow mapping:
- startup and resume rules map to `session-start.py`.
- new workflow start maps to `start-workflow.py`.
- planning maps to `planning`.
- development maps to `implementing`.
- tests map to the `testing` gate.
- reviews map to the `reviewing` gate.
- task completion commits map to `commit-task.py`.
- L0/L1 completion maps to `complete-workflow.py`.
- L2/L3 archive maps to `archive-plan.py`.
- incoming backlog intake maps to `backlog-intake.py`.
- backlog consumption maps to `backlog-consume.py`.

Truth sources:
- Workflow runtime: `work/workflow-state.json`
- Task runtime: `work/plans/active/<PLAN-ID>/tasks.json`
- Planning contract: `work/plans/active/<PLAN-ID>/plan.md`
- Recovery summary: `work/plans/active/<PLAN-ID>/handoff.md`
- Project environment contract: `.harness/contracts/project-contracts.json`
- Project entrypoint contract: `.harness/contracts/project-entrypoints.json`

Write gateways:
- `workflow-state.json` is written only through `state-write.py` or lifecycle tools that call it.
- `tasks.json` is initialized through `materialize-tasks.py` and updated through `update-task.py`.
- Phase transitions use `lifecycle-transaction.py` when available.
- Backlog intake writes use `backlog-intake.py`.
- Backlog consumption writes use `backlog-consume.py`.
- Entry point managed block updates must not re-run or overwrite `.harness/contracts/project-contracts.json`.

Task modeling:
- A task is a deliverable work unit.
- Testing, review, architecture impact, commit, and handoff are gates or audit actions, not tasks.
- L0/L1 do not create plans.
- L2/L3 use one active plan and one active task during implementing/testing/reviewing.
<!-- harness-engineering:end -->
