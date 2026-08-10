---
name: mlops-automation
description: Automate an MLOps project with mise tasks, lefthook hooks, Docker images, GitHub Actions, and MLflow tracking on a SQL backend. Use when adding a task runner, git hooks, CI/CD, or experiment tracking to a working package.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-automation
  created: 2026-01-25
  updated: 2026-08-10
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
   - `check`: Static checks in parallel (`ruff check`, `ty`, `pip-audit`, `gitleaks`, `trivy`, `actionlint` + `zizmor`).
   - `test`: Run `pytest`.
   - `build`: Build the wheel (`uv build`).
1. **The Gate**: define `all = ["mise run format", "mise run check", "mise run test", "mise run build"]`. This is the one command a developer, a hook, an agent, or CI runs; nothing else is allowed to define "ready".
1. **Pinned Tools**: declare every non-Python tool under `[tools]`, run `mise lock`, and commit `mise.lock` — it records the exact version, URL, and checksum per platform, so `uv.lock` pins the libraries and `mise.lock` pins the binaries.
1. **No Silent Installs**: set `[settings.task] run_auto_install = false` so a task fails loudly on a missing tool instead of downloading one mid-run.

### 2. Git Hooks

Catch issues locally with `lefthook` (replaces `pre-commit`).

1. **Framework**: `lefthook` with thin hooks — every command delegates to a `mise run` task so hooks and CI stay identical.
1. **Explicit Priorities**: lefthook orders commands **alphabetically**, so state the order yourself — formatters at 10, the staged secret scan at 20, the whole-tree checks at 30. Without priorities, `check` can read files that the formatter has not rewritten yet.
1. **Staged Files**: pass `{staged_files}` to the formatters and set `stage_fixed: true`, so a reformat is restaged into the commit being made rather than left dirty in the working tree.
1. **Secret Gate**: run `mise run check:leaks --staged` before the checks. A history scan does not look at the commit you are about to create; `--staged` does.
1. **pre-push**: Run `mise run test`.
1. **Example**:

   ```yaml
   # lefthook.yml
   pre-commit:
     parallel: false
     commands:
       format:dprint:
         priority: 10
         glob: "*.{json,md,toml,yaml,yml}"
         run: mise run format:dprint {staged_files}
         stage_fixed: true
       format:python:
         priority: 10
         glob: "*.py"
         run: mise run format:python {staged_files}
         stage_fixed: true
       check:leaks:
         priority: 20
         run: mise run check:leaks --staged
       check:
         priority: 30
         run: mise run check

   pre-push:
     commands:
       test:
         run: mise run test
   ```

1. **Commits**: Enforce **Conventional Commits** (e.g., `feat: add new model`) so `git-cliff` can generate the changelog.

### 3. Containerization

Reproducibility anywhere.

1. **Tool**: `docker`.
1. **Base Image**: Use `python:3.14-slim` for a minimal footprint; copy a **pinned** `uv` binary into the build stage (`ghcr.io/astral-sh/uv:0.12.3`, never `:latest` — a floating tag is invisible to Dependabot).
1. **Optimization**:
   - **Layer Caching**: Copy `uv.lock` + `pyproject.toml` and run `uv sync` _before_ copying `src/`.
   - **Multi-stage**: Build inputs in one stage, copy only artifacts (the resolved `.venv` or `dist/*.whl`) to the runtime stage.
1. **Non-root**: Run as a fixed numeric user (`USER 10001:10001`) and copy with `--chown=10001:10001`, so ownership is stable across rebuilds and readable by a host that does not share the image's `/etc/passwd`.
1. **Lint It**: add `check:dockerfile = "hadolint Dockerfile"` to `check`; the Dockerfile is code and deserves the same gate.
1. **Registry**: ask for the company artifact registry, or use `ghcr.io` for GitHub.

### 4. CI/CD Workflows

Automate verification and release with GitHub Actions.

1. **Platform**: ask for the company CI/CD platform, or use `github-actions` for GitHub.
1. **One Step**: CI runs `mise run all` and nothing else. A workflow that lists `format`, then `check`, then `test` as separate steps will drift from the local gate; a workflow with one step cannot.
1. **Hardening**: pin the runner (`ubuntu-24.04`), set `timeout-minutes`, keep `permissions: contents: read`, and check out with `persist-credentials: false` so no step inherits a push token it does not need.
1. **Prove Cleanliness**: end with `test -z "$(git status --porcelain)"`. Because `mise run all` starts by formatting, a dirty tree at the end means a contributor committed unformatted files.
1. **Example**:

   ```yaml
   # .github/workflows/ci.yml
   name: CI
   on:
     push:
       branches: [main]
     pull_request:
   permissions:
     contents: read
   concurrency:
     group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
     cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
   jobs:
     check:
       runs-on: ubuntu-24.04
       timeout-minutes: 20
       steps:
         - name: Checkout repository
           uses: actions/checkout@v7
           with:
             persist-credentials: false # no step pushes back to the repository
         - name: Install mise system
           uses: jdx/mise-action@v4
           with:
             cache: true # reuse the tools pinned by mise.lock
         - name: Run canonical gate
           run: mise run all
         - name: Verify no changes
           run: test -z "$(git status --porcelain)"
   ```

1. **Audit the Workflows**: add `check:actions = ["actionlint", "zizmor --offline .github/workflows/"]` to `check`. `actionlint` catches invalid syntax and shell bugs; `zizmor` catches workflow security issues (script injection, over-broad permissions, credential persistence). Commit `.github/zizmor.yml` to record the pinning policy the project actually follows, so the audit enforces it instead of fighting it.
1. **Scheduled Security Pass**: push CI only scans recent commits, so add `.github/workflows/security.yml` on a weekly `schedule` (plus `workflow_dispatch`) that checks out with `fetch-depth: 0` and runs the full-history `gitleaks git` and a full `trivy fs .`. Keep `fetch-depth: 0` **out** of `ci.yml`: a full clone on every push costs time and buys nothing the scheduled job does not already cover.
1. **Release**: `cd.yml` on Release builds the image and publishes docs via the official GitHub Pages Actions (`configure-pages`, `upload-pages-artifact`, `deploy-pages`).
1. **Dependencies**: add `.github/dependabot.yml` for every ecosystem in use (`uv`, `github-actions`, `docker`), grouping minor and patch updates into one pull request per ecosystem so majors are still reviewed alone.

### 5. AI/ML Experiments & Registry

Manage the ML lifecycle with **MLflow 3.15**.

1. **Platform**: `MLflow` (3.15+).
1. **Backend**: use a SQL store, locally as well as in production. `sqlite:///mlflow.db` is a real SQLAlchemy backend that supports the model registry and shares its shape with a production Postgres, so promotion is a `MLFLOW_TRACKING_URI` change rather than a rewrite. The **file store is deprecated**: it never supported the registry, and `MLFLOW_ALLOW_FILE_STORE=true` is an escape hatch to migrate off, not a configuration to ship.
1. **Local Server**: `mlflow server --backend-store-uri=sqlite:///mlflow.db --artifacts-destination=./mlartifacts`. Add `mlflow.db` to `.gitignore` — it is local state, and committing it puts a binary database in the history.
1. **Tracking**: Use `mlflow.autolog()`; log metrics, params, and artifacts.
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

### 7. Write It Down

Automation only helps if the next contributor finds it.

1. **`AGENTS.md`**: the commands, the definition of done, the conventions, and the layout — for AI assistants.
1. **`README.md`**: the same ground truth for humans, without the agent-facing rules.
1. **Rule**: when a task changes, both files change in the same commit. A stale command in `AGENTS.md` is worse than no command at all.

## Self-Correction Checklist

- [ ] **Task Vocabulary**: Does `mise run all` (and `format`/`check`/`test`/`build`) work?
- [ ] **Pinning**: Are `mise.lock` and `uv.lock` committed?
- [ ] **Hooks**: Do the hooks use explicit priorities, `{staged_files}`, and `check:leaks --staged`?
- [ ] **Image**: Is the Dockerfile multi-stage on `python:3.14-slim`, non-root, with a pinned `uv`, and does `hadolint` pass?
- [ ] **CI/CD**: Is `ci.yml` a single `mise run all` step with `persist-credentials: false` and the porcelain assertion?
- [ ] **Workflow Audit**: Do `actionlint` and `zizmor` pass, and does `security.yml` rescan the full history weekly?
- [ ] **Tracking**: Do runs land in a SQL-backed MLflow store, with validated thresholds before promotion?
