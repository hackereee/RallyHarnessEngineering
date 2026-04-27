# Handoff Template Design Note

Canonical handoff templates live under `.harness/templates/`.

- Runtime template: `.harness/templates/handoff.template.md`
- Runtime owner: L2/L3 active plan package under `work/plans/active/<PLAN-ID>/handoff.md`
- Truth sources: `work/workflow-state.json` and `work/plans/active/<PLAN-ID>/tasks.json`

Design copies must not invent a different state shape. After plan materialization and before lifecycle activation, `activeTaskId` is `null`, `currentPhase` is `planning`, `ownerRole` is `planner`, and all tasks in `tasks.json` remain `idle/developer`.
