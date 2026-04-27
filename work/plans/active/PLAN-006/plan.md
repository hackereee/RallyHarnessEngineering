# PLAN-006: Add Task Completion Commit Gate

## Background and Goal

The workflow lifecycle currently moves a task to `done` after verification and review pass, then continues to the next task or plan archive. It does not define a deterministic commit gate for the completed task. This plan adds a task completion commit operation so every finished task produces a Git commit before the next delivery work proceeds.

## Scope

- Document the task completion commit gate at the lifecycle point after `review-passed`.
- Allow the commit to include the state transition for activating the next task when that transition is part of closing the current task.
- Provide a deterministic script and CLI command for committing a completed task.
- Validate that only reviewed, verified, `done` tasks can pass the commit gate.
- Add regression tests before implementation.

## Non-Scope

- Do not model commit, testing, or review as standalone tasks.
- Do not make `review-passed` automatically run `git commit`.
- Do not add commit hashes to `tasks.json` as task truth; Git history is the audit source for commits.
- Do not change archive storage layout.

## Implementation Direction

Keep `review-passed` responsible for lifecycle state transitions and postflight validation. Add a separate deterministic commit gate script that checks the workflow and task truth sources, stages the current worktree, commits with a task-scoped message, and reports the resulting commit. The script must allow the worktree to contain follow-on lifecycle state changes, such as activating the next task, because those changes can be part of closing the completed task before the next implementation starts.

## File Boundaries

- Create: `.harness/scripts/commit-task.py`
- Modify: `.harness/scripts/harness`
- Modify: `.harness/scripts/session-start.py`
- Modify: `.harness/scripts/lifecycle-transaction.py`
- Modify: `.harness/rules/workflow-lifecycle.md`
- Modify: `.harness/rules/archive-rules.md`
- Modify: `harness-design/architecture.md`
- Test: `.harness/tests/test_commit_task.py`
- Test: `.harness/tests/test_harness_cli.py`
- Test: `.harness/tests/test_session_start.py`
- Test: `.harness/tests/test_lifecycle_transaction.py`

## Task Decomposition

This plan uses one delivery task because the commit gate is a single lifecycle contract. The rule text, script behavior, CLI surface, and regression tests must remain consistent.

## Verification Strategy

Use TDD for the new commit script and lifecycle output contract, then run the related CLI/session/lifecycle tests. Finish with the full Harness test suite, lint, and current workflow-state validation.

## Risks and Open Questions

- Risk: Automatically staging the whole worktree can include unrelated edits if the operator violates the workflow discipline. The script should print the staged paths and reject an empty diff, while the workflow rule requires the gate to be run before unrelated work starts.
- Risk: Requiring a commit after every done task may add extra commits for small L0/L1 direct workflows. This plan focuses on L2/L3 task commits; L0/L1 can use normal workflow completion commits outside this task-specific gate.
- Open questions: None blocking. The user explicitly approved including next-task activation state changes in the commit.

## Plan Review Gate

Status: passed
Reviewer: harness-reviewer
Reviewed At: 2026-04-27T22:00:00+08:00

Checks:
- Scope, non-scope, file boundaries, dependencies, acceptance, and verification are reviewable.
- The plan contains one delivery task and no commit-only, testing-only, or review-only tasks.
- The commit gate is placed after task review passes and before new implementation work starts.
- The design keeps scripts deterministic and keeps semantic closure in the lifecycle rules.

Findings:
- No blocking findings.

## Task Contracts

<a id="task-001-add-task-completion-commit-gate"></a>

### TASK-001: Add task completion commit gate

Goal: Add a deterministic workflow gate that commits each verified and reviewed completed task before subsequent implementation work proceeds.

Files:
- Create: `.harness/scripts/commit-task.py`
- Modify: `.harness/scripts/harness`
- Modify: `.harness/scripts/session-start.py`
- Modify: `.harness/scripts/lifecycle-transaction.py`
- Modify: `.harness/rules/workflow-lifecycle.md`
- Modify: `.harness/rules/archive-rules.md`
- Modify: `harness-design/architecture.md`
- Test: `.harness/tests/test_commit_task.py`
- Test: `.harness/tests/test_harness_cli.py`
- Test: `.harness/tests/test_session_start.py`
- Test: `.harness/tests/test_lifecycle_transaction.py`

Depends on: []

Acceptance:
- The lifecycle rule states that `review-passed` creates a required task completion commit gate before new implementation work.
- The commit gate may include next-task activation state changes when the completed task is the one being committed.
- The script rejects tasks that are not `done`, lack passed verification, lack passed review, or still have blocking review findings.
- The script rejects an empty worktree diff and commits staged work with a task-scoped Chinese default message.
- The CLI exposes the commit gate as a first-class Harness command.
- Session startup treats the commit gate script as a required Harness asset.

Verification:
- Run: `python3 .harness/tests/test_commit_task.py`
- Run: `python3 .harness/tests/test_harness_cli.py`
- Run: `python3 .harness/tests/test_session_start.py`
- Run: `python3 .harness/tests/test_lifecycle_transaction.py`
- Run: `python3 -m unittest discover -s .harness/tests -p 'test_*.py'`
- Run: `python3 .harness/scripts/lint-harness.py --root .`
- Run: `python3 .harness/scripts/validate-state.py --state work/workflow-state.json --schema .harness/schemas/workflow-state.schema.json`
