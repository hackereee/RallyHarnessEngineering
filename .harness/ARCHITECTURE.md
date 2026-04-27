# Harness Framework Architecture

This document describes the reusable Harness framework installed under `.harness/`. It is framework architecture, not the target project's business architecture.

## Layers

```text
repo/
├─ AGENTS.md or another agent entrypoint
├─ ARCHITECTURE.md              # optional business architecture owned by the target project
├─ .harness/                    # reusable Harness framework assets
│  ├─ ARCHITECTURE.md           # this framework architecture document
│  ├─ schemas/                  # machine-checkable contracts
│  ├─ templates/                # initialization templates
│  ├─ contracts/                # project-specific contracts
│  ├─ rules/                    # semantic rules that schemas cannot express
│  ├─ skills/                   # repo-local agent workflow skills
│  ├─ scripts/                  # deterministic gateways and validators
│  └─ tests/                    # framework regression tests
└─ work/                        # runtime state, plans, handoff, and session audit data
```

## Core Boundaries

- `.harness/` stores contracts, templates, rules, skills, scripts, tests, and this framework architecture document.
- `work/` stores runtime data such as `workflow-state.json`, active plan packages, archived plans, and session audit records.
- `workflow-state.json` is the workflow-level runtime truth source.
- `work/plans/active/<PLAN-ID>/tasks.json` is the task-level runtime truth source.
- `handoff.md` and session files are recovery and audit evidence, not truth sources.
- Project-specific environment facts belong in `.harness/contracts/project-contracts.json`.
- Project agent entrypoint facts belong in `.harness/contracts/project-entrypoints.json`.
- Root `ARCHITECTURE.md` belongs to the target project's business architecture. Project initialization creates it as an empty file when missing so later task completion summaries have a stable place to update business architecture conclusions.

## Lifecycle

Harness routes work through workflow phases:

```text
planning -> implementing -> testing -> reviewing -> archiving
```

Testing, review, and Architecture Impact are workflow gates, not standalone tasks. Architecture Impact records whether root `ARCHITECTURE.md` or Harness framework architecture needs an update after the delivered change. L2/L3 workflows use one active plan and at most one active task during execution phases. Existing `workflow-state.json` updates must go through `state-write.py`; task state updates must go through `update-task.py`.

## Real Project Integration

The target project's agent entrypoint should reference this document through a managed block. The root `ARCHITECTURE.md`, when present, remains the target project's business architecture. Do not paste Harness framework architecture into root `ARCHITECTURE.md`.
