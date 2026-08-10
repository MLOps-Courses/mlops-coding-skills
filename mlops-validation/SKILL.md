---
name: mlops-validation
description: Add the validation layers that gate a merge — ty typing, Ruff linting, pytest coverage, structured logging, and the trivy, pip-audit, and gitleaks scans. Use when hardening code quality or wiring the mise run check task.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-validation
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Validation

## Goal

To ensure software quality, reliability, and security through automated validation layers. This skill enforces **Strict Typing** (`ty`), **Unified Linting** (`ruff`), **Comprehensive Testing** (`pytest`), **Structured Logging**, and **Supply-Chain Scanning** (`pip-audit`, `gitleaks`, `trivy`).

## Prerequisites

- **Language**: Python 3.14
- **Manager**: `uv`; **Tasks**: `mise`
- **Context**: Ensuring code quality before merge/deploy.

## Instructions

### 1. The Task Vocabulary

Every check below is a `mise` task, so the same command runs locally, in the git hook, and in CI. Never invoke the underlying tool by hand in a hook or a workflow — the task is the single definition.

| Task            | Tool                                                                     | What it proves                                                        |
| --------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| `check:format`  | `dprint`, `validate-pyproject`, `ruff format --check`, `uv lock --check` | Files and manifests are canonical and the lockfile is current.        |
| `check:lint`    | `ruff check`                                                             | No lint violation, including the `S` (security) rules.                |
| `check:types`   | `ty check`                                                               | Type annotations are consistent.                                      |
| `check:vuln`    | `pip-audit`                                                              | No known CVE in the resolved Python dependencies.                     |
| `check:leaks`   | `gitleaks`                                                               | No secret in the staged change or the recent history.                 |
| `check:scan`    | `trivy`                                                                  | No misconfiguration, leaked secret, or forbidden license in the tree. |
| `check:actions` | `actionlint`, `zizmor`                                                   | Workflows are valid and not vulnerable.                               |
| `test`          | `pytest`                                                                 | Behavior is covered and correct.                                      |

`mise run check` runs the `check:*` tasks in parallel; `mise run all` chains `format` -> `check` -> `test` -> `build` and is the only gate anyone needs to remember.

### 2. Static Analysis (Typing & Linting)

Catch errors before they run.

1. **Typing**:
   - **Tool**: `ty` (Astral type checker; pre-1.0, pin a compatible range such as `ty>=0.0.69,<0.1`). The mandated checker — do not use `mypy`.
   - **Rule**: No `Any` (unless absolutely necessary). Fully typed function signatures.
   - **Pragmatism**: `ty` does not yet model every dynamic library (MLflow, pandera, Pydantic). Silence the specific rule categories they trigger in `[tool.ty.rules]`, with a comment saying why — never disable the checker wholesale.
   - **DataFrames**: Use `pandera` schemas to validate DataFrame structures/types.
   - **Classes**: Use `pydantic` for data modeling and runtime validation.
1. **Linting & Formatting**:
   - **Tool**: `ruff` 0.16+ (replaces black, isort, pylint, flake8, bandit).
   - **Rule**: Zero tolerance for linter errors. Use `noqa` sparingly and with justification.
   - **Config**: Centralize in `pyproject.toml`, with an explicit `[tool.ruff.lint] select` list. Ruff 0.16 expanded the _default_ rule set from 59 to 413 rules, so a project that relies on the default set changes behavior on upgrade while an explicit `select` list does not.
   - **Markdown**: Ruff 0.16 also formats Python code blocks inside `.md` files. Expect a one-time reformat of your documentation on the first run, and commit it.

### 3. Testing Strategy

Verify behavior and prevent regressions.

1. **Tool**: `pytest` (9.x).
1. **Structure**: Mirror `src/` in `tests/`.

   ```text
   src/pkg/mod.py -> tests/test_mod.py
   ```

1. **Fixtures**: Use `tests/conftest.py` for shared setup (mock data, temp paths).
1. **Coverage**: Measure with `pytest-cov` and set `--cov-fail-under` to the level the suite actually reaches, so any drop is a visible regression rather than slack under a round number.
1. **Pattern**: Use **Given-When-Then** in comments.

   ```python
   def test_pipeline_execution(input_data):
       # Given: Valid input data
       # When: The pipeline processes the data
       # Then: The output content matches expectations
   ```

1. **MLflow in tests**: Point the tests at a SQLite tracking store, not the deprecated file store — but build it once. Creating a fresh MLflow SQLite database runs the full Alembic migration chain (measured at roughly 7 seconds per database), so a session-scoped fixture should migrate one template database and each test should `shutil.copyfile` it into its own `tmp_path` (roughly 0.07 seconds). Migrating per test turned a 34-second suite into a 339-second one.

### 4. Structured Logging

Enable observability and debugging.

1. **Tool**: `loguru`, configured through a small logging service so sinks and levels stay configurable. The wider house standard for long-lived services is `structlog`; both emit structured records, so choose one per project and stay with it — running both splits the log stream and doubles the configuration surface.
1. **Format**: Use structured logging (JSON) in production for queryability.
1. **Levels**:
   - `DEBUG`: Low-level tracing (payloads, internal state).
   - `INFO`: Key business events (Job started, Model saved).
   - `ERROR`: Actionable failures (with stack traces).
1. **Context**: Include context (Job ID, Model Version) in logs.
1. **Discipline**: No bare `print` in library code — Ruff's `T20` rules enforce it.

### 5. Security

Protect the supply chain and runtime.

1. **Code Scanning**: Enable Ruff `S` (flake8-bandit) rules to detect unsafe patterns (e.g., `eval`, `yaml.load`) — this replaces standalone `bandit`, and runs inside `check:lint`.
1. **Dependencies**: `check:vuln` runs `pip-audit --skip-editable` against the resolved environment; **Dependabot** opens the update pull requests on a weekly schedule.
1. **Secret Scanning**: `check:leaks` runs `gitleaks`. Scan the staged change in the pre-commit hook (`--staged`) and the recent history in CI (`--log-opts="--max-count=100"`); a scheduled workflow rescans the **full** history weekly, because a secret committed and later removed is invisible to a shallow scan forever.
1. **Filesystem Scanning**: `check:scan` runs `trivy --config trivy.yaml fs .` — one pass covering vulnerabilities, misconfigurations (Dockerfile, IaC), secrets, and license compliance. Two details matter: pass `--config` explicitly, or a `TRIVY_CONFIG` exported in a developer's shell silently overrides the committed policy; and use the `fs` subcommand, because `trivy config` only runs the misconfiguration scanner and quietly skips the rest.
1. **Workflow Scanning**: `check:actions` runs `actionlint` (syntax, shell) and `zizmor` (workflow security: injection, over-broad permissions, credential persistence).
1. **Secrets**: **NEVER** log secrets. Sanitize outputs.

## Self-Correction Checklist

- [ ] **Type Safety**: Does `mise run check:types` pass?
- [ ] **Lint Cleanliness**: Does `mise run check:lint` pass with an explicit `select` list?
- [ ] **Test Discovery**: Does `pytest` successfully find modules in `src/`?
- [ ] **Test Speed**: Is the MLflow store built once and copied, not migrated per test?
- [ ] **Log Format**: Are production logs serializing to JSON, from one logging library?
- [ ] **Security**: Do `check:lint` (Ruff `S`), `check:vuln`, `check:leaks`, `check:scan`, and `check:actions` all pass?
- [ ] **Gate**: Does `mise run all` pass end to end?
