---
name: project-init
description: Use when initializing Harness in a real project by detecting agent entrypoints, installing Harness architecture references, and delegating environment contracts.
---

# Project Init

## Overview

Initialize Harness in a real project. This top-level skill connects the target repository's agent entrypoint to the stable Harness framework architecture and then delegates project environment contract derivation to `project-env-contract`.

This skill is about onboarding and coordination. It must not replace deterministic scripts, write workflow runtime state directly, or turn project-specific environment checks into Harness core startup checks.

## Entrypoint Detection

Inspect repository evidence before writing references. Detect agent entrypoint candidates in this order:

1. Canonical generic entrypoint:
   - `AGENTS.md`
2. Tool-specific entrypoints:
   - `CLAUDE.md`
   - `GEMINI.md`
   - `.github/copilot-instructions.md`
3. Editor or agent rule files:
   - `.cursor/rules/*.mdc`
   - `.cursorrules`
   - `.windsurfrules`
   - `.windsurf/rules/*.md`
   - `.clinerules`
   - `.roo/rules/*.md`

If `AGENTS.md` exists, use it as the canonical entrypoint. If multiple tool-specific entrypoints exist without `AGENTS.md`, report the candidates and recommend creating `AGENTS.md` as the canonical entrypoint unless the user explicitly chooses another entrypoint.

If no agent entrypoint exists, report `NEEDS_ENTRYPOINT` and recommend creating `AGENTS.md` unless the user explicitly chooses another entrypoint. Do not silently write to `README.md`, `CONTRIBUTING.md`, or root `ARCHITECTURE.md`; those are human-facing fallback evidence, not agent entrypoints by default.

## Architecture Reference

Harness framework architecture belongs at `.harness/ARCHITECTURE.md`.

The target repository's root `ARCHITECTURE.md`, when present, remains business architecture. Do not paste the Harness framework architecture into root `ARCHITECTURE.md`. The entrypoint should reference both documents with separate meanings:

- root `ARCHITECTURE.md`: business architecture, modules, runtime topology, and project boundaries;
- `.harness/ARCHITECTURE.md`: Harness framework architecture, lifecycle, schemas, scripts, rules, skills, and `work/` runtime layout.

If `.harness/ARCHITECTURE.md` is missing, install it from the Harness framework assets before inserting entrypoint references.

## Managed Block Update

Entrypoint changes must go through a deterministic managed block, not freeform rewriting. The block markers are:

```md
<!-- harness-engineering:start -->
<!-- harness-engineering:end -->
```

The managed block must tell future agents to read:

1. the current agent entrypoint;
2. root `ARCHITECTURE.md` if present;
3. `.harness/ARCHITECTURE.md`;
4. `.harness/rules/workflow-lifecycle.md`;
5. `.harness/contracts/`.

It must also name the truth sources:

- workflow runtime: `work/workflow-state.json`;
- task runtime: `work/plans/active/<PLAN-ID>/tasks.json`;
- project environment contract: `.harness/contracts/project-contracts.json`;
- project entrypoint contract: `.harness/contracts/project-entrypoints.json`.

Do not modify content outside the managed block. If a managed block already exists, replace that block only.

## Environment Contract Delegation

After the entrypoint and Harness architecture reference are configured, delegate project environment contract derivation to `project-env-contract`.

The delegated output is `.harness/contracts/project-contracts.json`. This skill may verify that the file exists or report that it remains `NOT_CONFIGURED`, but it must not duplicate the `project-env-contract` rules such as project profile extraction, command registry construction, environment check severity mapping, or adapter fallback policy.

## Output Boundary

This skill may guide the Agent to:

- choose or create an agent entrypoint;
- install or verify `.harness/ARCHITECTURE.md`;
- run the deterministic entrypoint updater;
- create or update `.harness/contracts/project-entrypoints.json`;
- invoke `project-env-contract` for `.harness/contracts/project-contracts.json`.

This skill must not:

- write `workflow-state.json` directly;
- write `tasks.json` directly;
- activate tasks or change workflow phase;
- add project-specific checks to `session-start.py`;
- rewrite root `ARCHITECTURE.md` with Harness framework architecture;
- mutate user entrypoint prose outside the managed block.

## Validation

Before claiming initialization is ready, verify:

- an agent entrypoint exists or the user explicitly chose to create one;
- the managed block is present exactly once in the canonical entrypoint;
- `.harness/ARCHITECTURE.md` exists and is referenced by the entrypoint;
- `.harness/contracts/project-entrypoints.json` records the canonical entrypoint;
- `project-env-contract` has either produced `.harness/contracts/project-contracts.json` or intentionally remains `NOT_CONFIGURED`;
- Harness core checks remain separate from project environment checks.
