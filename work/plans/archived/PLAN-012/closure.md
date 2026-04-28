# Closure

- workflowId: workflow-plan-012-v1
- planId: PLAN-012
- result: completed
- archivedAt: 2026-04-28T14:12:00+08:00

## Delivered

- Added `installer/release/check_artifacts.py` to verify local release artifacts before publication.
- Added `installer/release/smoke_install.py` to install the built wheel in a temporary virtual environment and exercise the installed `harness-engineering` CLI.
- Added `.github/workflows/publish-python-package.yml` as a manual TestPyPI/PyPI release workflow using PyPI Trusted Publishing.
- Added `docs/release/package-registry-release.md` with registry setup, TestPyPI validation, PyPI promotion, install/upgrade commands, and yank/rollback guidance.
- Updated README and installer lifecycle documentation to expose release gates and operator docs.
- Added release regression tests under `installer/tests/`.

## Verification Evidence

- `python3 installer/tests/test_release_artifacts.py`
- `python3 installer/tests/test_release_smoke.py`
- `python3 installer/tests/test_release_docs.py`
- `python3 -m unittest discover -s installer/tests -p 'test_*.py'`
- `python3 -m build`
- `python3 installer/release/check_artifacts.py dist`
- `python3 installer/release/smoke_install.py dist`
- `python3 .harness/scripts/lint-harness.py --root .`
- `python3 .harness/scripts/harness validate-state`

## Review Summary

- TASK-001 passed review with score 94/85 and no blocking findings.
- TASK-002 passed review with score 93/85 and no blocking findings.
- TASK-003 passed review with score 92/85 and no blocking findings.
- Testing and review remained workflow gates, not standalone tasks.
- All task completion commits were created before this closure.

## Architecture Impact

- Target project architecture: root `ARCHITECTURE.md` is absent and was not introduced; release operation details belong in README and `docs/release/package-registry-release.md`.
- Harness framework architecture: `.harness/ARCHITECTURE.md` remains unchanged because package registry publishing is installer/distribution automation outside the `.harness/` runtime framework boundary.
- Installer lifecycle: `installer/install-lifecycle.md` was updated to record package release gates and link the manual publishing workflow and registry operation guide.

## Deviations

- `.gitignore` was updated during TASK-001 to ignore Python packaging outputs (`build/`, `dist/`, `*.egg-info/`). This was necessary to keep local verification artifacts out of Harness task commits.
- `smoke_install.py` installs the local wheel with `pip install --no-deps`; dependency metadata is verified by `check_artifacts.py`, while smoke testing remains deterministic and avoids network dependency resolution.

## Follow-ups

- Real TestPyPI/PyPI publication still requires explicit operator instruction and external Trusted Publisher configuration.
- Before publishing, bump `pyproject.toml` version and run the documented release gates.
