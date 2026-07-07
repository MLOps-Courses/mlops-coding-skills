---
name: mlops-validation
description: Guide to implement rigorous validation layers including static analysis, automated testing, structured logging, and security scanning.
---

# MLOps Validation

## Goal

To ensure software quality, reliability, and security through automated validation layers. This skill enforces **Strict Typing** (`ty`), **Unified Linting** (`ruff`), **Comprehensive Testing** (`pytest`), and **Structured Logging** (`loguru`).

## Prerequisites

- **Language**: Python
- **Manager**: `uv`
- **Context**: Ensuring code quality before merge/deploy.

## Instructions

### 1. Static Analysis (Typing & Linting)

Catch errors before they run.

1. **Typing**:
    - **Tool**: `ty` (Astral type checker; pre-1.0, pin a compatible range). The mandated checker — do not use `mypy`.
    - **Rule**: No `Any` (unless absolutely necessary). Fully typed function signatures.
    - **DataFrames**: Use `pandera` schemas to validate DataFrame structures/types.
    - **Classes**: Use `pydantic` for data modeling and runtime validation.
2. **Linting & Formatting**:
    - **Tool**: `ruff` (replaces black, isort, pylint, flake8).
    - **Rule**: Zero tolerance for linter errors. Use `noqa` sparingly and with justification.
    - **Config**: Centralize in `pyproject.toml`.

### 2. Testing Strategy

Verify behavior and prevent regressions.

1. **Tool**: `pytest`.
2. **Structure**: Mirror `src/` in `tests/`.

    ```text
    src/pkg/mod.py -> tests/test_mod.py
    ```

3. **Fixtures**: Use `tests/conftest.py` for shared setup (mock data, temp paths).
4. **Coverage**: Aim for high coverage (>80%) on core business logic. Use `pytest-cov`.
5. **Pattern**: Use **Given-When-Then** in comments.

    ```python
    def test_pipeline_execution(input_data):
        # Given: Valid input data
        # When: The pipeline processes the data
        # Then: The output content matches expectations
    ```

### 3. Structured Logging

Enable observability and debugging.

1. **Tool**: `loguru` (replacing stdlib `logging`).
2. **Format**: Use structured logging (JSON) in production for queryability.
3. **Levels**:
    - `DEBUG`: Low-level tracing (payloads, internal state).
    - `INFO`: Key business events (Job started, Model saved).
    - `ERROR`: Actionable failures (with stack traces).
4. **Context**: Include context (Job ID, Model Version) in logs.

### 4. Security

Protect the supply chain and runtime.

1. **Code Scanning**: Enable Ruff `S` (flake8-bandit) rules to detect unsafe patterns (e.g., `eval`, `yaml.load`) — this replaces standalone `bandit`.
1. **Dependencies**: Run `pip-audit` (and/or `GitHub Dependabot`) to patch vulnerable packages.
1. **Secret Scanning**: Run `gitleaks` to keep credentials out of the code and git history.
1. **Secrets**: **NEVER** log secrets. Sanitize outputs.

## Self-Correction Checklist

- [ ] **Type Safety**: Does `ty` pass without errors?
- [ ] **Lint Cleanliness**: Does `ruff check` pass?
- [ ] **Test Discovery**: Does `pytest` successfully find modules in `src/`?
- [ ] **Log Format**: Are production logs serializing to JSON?
- [ ] **Security**: Do Ruff `S` rules, `pip-audit`, and `gitleaks` pass?
