---
name: mlops-automation
description: Guide to refine MLOps projects with task automation, containerization, CI/CD pipelines, and robust experiment tracking.
---

# MLOps Automation

## Goal

To elevate the codebase to production standards by adding **Task Automation** (`mise`), **Git Hooks** (`lefthook`), **Containerization** (`docker`), **CI/CD** (`github-actions`), and **Experiment Tracking** (`mlflow`).

## Prerequisites

- **Language**: Python 3.14
- **Manager**: `uv`
- **Context**: Preparing for scale and deployment.

## Instructions

### 1. Task Automation

Expose a single, shared task vocabulary with `mise` (replaces `just`/`make`).

1. **Tool**: `mise` — pins the toolchain and defines tasks in `mise.toml`.
1. **Vocabulary**: `install`, `format`, `check`, `test`, `build`, `watch`. Run everything via `mise run <task>` so hooks and CI reuse the same entrypoints.
1. **Core Tasks**:
    - `format`: Format code and config (`ruff format`, `dprint fmt`).
    - `check`: Static checks (`ruff check`, `ty`, security rules).
    - `test`: Run `pytest`.
    - `build`: Build the wheel (`uv build`).

### 2. Git Hooks

Catch issues locally with `lefthook` (replaces `pre-commit`).

1. **Framework**: `lefthook` with thin hooks — every command delegates to a `mise run` task so hooks and CI stay identical.
1. **pre-commit**: Run `mise run format` then `mise run check`.
1. **pre-push**: Run `mise run test`.
1. **Security**: Prefer Ruff `S` rules (replaces `bandit`) plus `pip-audit`/`gitleaks` (see the Validation skill), not a separate scanner.
1. **Commits**: Enforce **Conventional Commits** (e.g., `feat: add new model`) so `git-cliff` can generate the changelog.

### 3. Containerization

Reproducibility anywhere.

1. **Tool**: `docker`.
1. **Base Image**: Use `python:3.14-slim` for a minimal footprint; install `uv` in the build stage.
1. **Optimization**:
    - **Layer Caching**: Copy `uv.lock` + `pyproject.toml` and run `uv sync` *before* copying `src/`.
    - **Multi-stage**: Build inputs in one stage, copy only artifacts (`dist/*.whl`) to the runtime stage.
1. **Registry**: ask for the company artifact registry, or use `ghcr.io` for GitHub.

### 4. CI/CD Workflows

Automate verification and release with GitHub Actions.

1. **Platform**: ask for the company CI/CD platform, or use `github-actions` for GitHub.
1. **Toolchain**: Bootstrap every job with `actions/checkout@v7` + `jdx/mise-action@v4` so CI runs the exact same `mise run` tasks as local hooks.
1. **Workflows**:
    - `ci.yml`: On push/PR, run `mise run format`, `mise run check`, `mise run test`.
    - `cd.yml`: On Release, build the image and publish docs via the official GitHub Pages Actions (`configure-pages`, `upload-pages-artifact`, `deploy-pages`).
1. **Optimization**: Use `concurrency` to cancel redundant runs.

### 5. AI/ML Experiments & Registry

Manage the ML lifecycle with **MLflow 3**.

1. **Platform**: `MLflow` (v3).
1. **Tracking**:
    - Use `mlflow.autolog()`; log metrics, params, and artifacts.
    - The file store is deprecated — opt in for local runs with `MLFLOW_ALLOW_FILE_STORE=true`; use a real backend (SQL/HTTP) in production.
1. **Models**: Log models with the keyword `name=` (e.g., `mlflow.pyfunc.log_model(name=...)`).
1. **Validation**: Gate promotion with `mlflow.validate_evaluation_results` against explicit metric thresholds.
1. **Registry**:
    - Register top models manually or via CI.
    - **Aliases**: Use `@champion` or `@production` for stable deployment pointers. Never rely on moving versions (e.g., `v1` -> `v2`).

### 6. Design Patterns

Write flexible code.

1. **Strategy**: For swappable algorithms (e.g., different model types).
1. **Factory**: For creating objects from config (e.g., `ModelFactory`).
1. **Adapter**: For standardizing mismatched interfaces.

## Self-Correction Checklist

- [ ] **Task Vocabulary**: Does `mise run check` (and `format`/`test`/`build`) work?
- [ ] **Hooks**: Do `lefthook` hooks delegate to `mise run` tasks?
- [ ] **Image**: Is the Dockerfile multi-stage on `python:3.14-slim`?
- [ ] **CI/CD**: Do `ci.yml`/`cd.yml` bootstrap with `jdx/mise-action@v4`?
- [ ] **Tracking**: Are runs logged to MLflow with validated thresholds?
