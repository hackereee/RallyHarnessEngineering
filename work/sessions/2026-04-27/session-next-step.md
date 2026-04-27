# Session Next Step

## Context

- Repo: LearnHarnessEngineering
- Date: 2026-04-27
- Recent completed commits:
  - `3f9ee3d` 新增生命周期流转事务协调器
  - `b321a38` 迁移 Harness 测试目录
  - `60d6c5b` 补齐归档闭环
- Current lifecycle coverage:
  - `session-start.py` handles session bootstrap and audit snapshot.
  - `lifecycle-transaction.py` coordinates `activate-next`, `start-testing`, `start-review`, `review-failed`, and `review-passed`.
  - `archive-plan.py` closes `archiving -> archived` after Agent-written `closure.md`.
  - Tests live under `.harness/tests/`.

## Decision

The best next engineering action is to implement the unified CLI entrypoint:

```text
.harness/scripts/harness <subcmd>
```

This should come before backlog schema, review block, or handoff-rules work.

## Rationale

- `harness-design/architecture.md` already states the invariant: external script usage should eventually go through `.harness/scripts/harness <subcmd>`.
- The lifecycle is now functionally closed, but the user still has to remember several individual script names.
- A unified entrypoint reduces accidental bypass of `lifecycle-transaction.py` and `archive-plan.py`.
- Backlog management is an intake concern; review block is evidence enrichment. Both are lower priority than making the existing lifecycle safe and ergonomic to invoke.

## Proposed Scope

Implement a thin dispatcher first, not a new framework:

- Add `.harness/scripts/harness`.
- Add `.harness/tests/test_harness_cli.py`.
- Route subcommands to existing scripts without reimplementing their logic.
- Start with:
  - `harness lint`
  - `harness validate-state`
  - `harness session-start`
  - `harness transition <action>`
  - `harness archive-plan <PLAN-ID>`
- Preserve exit codes and stdout/stderr from delegated scripts.
- Update `architecture.md`, `AGENTS.md`, and relevant tests.

## Next Action

Write a failing `.harness/tests/test_harness_cli.py` covering `harness lint` and `harness transition --help`.
