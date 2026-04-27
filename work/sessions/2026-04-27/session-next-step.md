# Session Next Step

## Context

- Repo: LearnHarnessEngineering
- Date: 2026-04-27
- Recent completed commits:
  - `3f9ee3d` 新增生命周期流转事务协调器
  - `b321a38` 迁移 Harness 测试目录
  - `60d6c5b` 补齐归档闭环
  - `93c2c58` 补齐 L0/L1 workflow 收口
- Current lifecycle coverage:
  - `session-start.py` handles session bootstrap and audit snapshot.
  - `lifecycle-transaction.py` coordinates `activate-next`, `start-testing`, `start-review`, `review-failed`, and `review-passed`.
  - `archive-plan.py` closes L2/L3 `archiving -> archived` after Agent-written `closure.md`.
  - `complete-workflow.py` closes L0/L1 `reviewing -> completed` with session audit evidence.
  - Tests live under `.harness/tests/`.

## Decision

The best next engineering action is to implement the unified CLI entrypoint:

```text
.harness/scripts/harness <subcmd>
```

This should come before backlog schema, review block enrichment, or handoff-rules work.

## Rationale

- `harness-design/architecture.md` already states the invariant: external script usage should eventually go through `.harness/scripts/harness <subcmd>`.
- The lifecycle is now closed for both direct workflows and plan-backed workflows, but users still need to remember several individual script names.
- A unified entrypoint reduces accidental bypass of `lifecycle-transaction.py`, `archive-plan.py`, `complete-workflow.py`, and the `state-write.py` gateway.
- Backlog management is intake; review block and handoff-rules are evidence enrichment. The current higher leverage step is making the existing lifecycle safe and ergonomic to invoke.

## Proposed Scope

Implement a thin dispatcher first, not a new framework:

- Add `.harness/scripts/harness`.
- Add `.harness/tests/test_harness_cli.py`.
- Route subcommands to existing scripts without reimplementing lifecycle logic.
- Start with:
  - `harness lint`
  - `harness validate-state`
  - `harness session-start`
  - `harness transition <action>`
  - `harness archive-plan <PLAN-ID>`
  - `harness complete-workflow ...`
- Preserve exit codes and stdout/stderr from delegated scripts.
- Ensure `harness validate-state` defaults to `work/workflow-state.json`, not `.harness/templates/workflow-state.template.json`.
- Update `architecture.md`, `session-start.py`, `AGENTS.md`, and relevant tests.

## Next Action

Write a failing `.harness/tests/test_harness_cli.py` covering `harness lint`, `harness transition --help`, and `harness validate-state` defaulting to `work/workflow-state.json`.
