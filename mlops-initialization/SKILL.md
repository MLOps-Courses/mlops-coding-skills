---
name: mlops-initialization
description: Initialize a production-ready Python MLOps project with uv, git, mise, and a shared editor setup. Use when starting a new repository, writing its first pyproject.toml, or repairing an inconsistent project skeleton.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-initialization
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Initialization

## Goal

To initialize a robust, production-ready MLOps project structure using the modern Python toolchain (`uv`), industry-standard version control (`git`), a shared task runner (`mise`), and a configured development environment (`VS Code`). This skill ensures reproducibility, collaboration, and high code quality from day one.

## Prerequisites

- **Language**: Python 3.14 (latest stable)
- **Manager**: `uv` (replaces pip, venv, poetry, pyenv)
- **Tasks**: `mise` (replaces make/just, and pins the toolchain)
- **VCS**: Git
- **IDE**: VS Code (recommended)

## Instructions

### 1. System & Toolchain Verification

Before modifying files, verify that the essential tools are available.

1. **Check `uv`**:
   - Ensure `uv` is installed: `uv --version`
   - If missing, install it: `curl -LsSf https://astral.sh/uv/install.sh | sh`
1. **Check `git`**:
   - Ensure `git` is installed: `git --version`
1. **Check `mise`**:
   - Ensure `mise` is installed: `mise --version`
   - `mise` pins every non-Python tool (`dprint`, `gitleaks`, `trivy`, `actionlint`, ...) so contributors and CI resolve identical binaries.

### 2. Project Initialization

Initialize the project structure using `uv` to ensure modern standards (`pyproject.toml`).

1. **Create Directory** (if not already inside):
   - `mkdir <project_name> && cd <project_name>`
1. **Initialize Project**:
   - Run `uv init`
   - This creates `pyproject.toml`, `.python-version`, and a basic `hello.py`.
1. **Configure `pyproject.toml`**:
   - Update **metadata**: `name`, `version`, `description`, `authors`, `license`.
   - Set **requires-python**: Ensure it matches the project's target environment (e.g., `>=3.14`).
   - Declare the license the **PEP 639** way: `license` is an SPDX expression (a plain string), and the file itself is listed in `license-files`. The old `license = { file = "LICENSE" }` table form is deprecated and rejected by current build backends.
   - **Example Structure**:

     ```toml
     [project]
     name = "my-mlops-project"
     version = "0.1.0"
     description = "A robust MLOps project."
     readme = "README.md"
     requires-python = ">=3.14"
     license = "MIT" # SPDX expression (PEP 639)
     license-files = ["LICENSE.txt"] # the file(s) shipped in the distribution
     authors = [{ name = "Your Name", email = "your.email@example.com" }]
     dependencies = [
       "loguru>=0.7.3",
       "mlflow>=3.15.1",
       # MLflow 3.15 still declares `pandas<3`, so the pandas 3.x line is unreachable
       # here; keep the floor permissive and let `uv.lock` pin the tested version.
       "pandas>=2.3.3",
       "pydantic>=2.13.4",
       "scikit-learn>=1.9.0",
     ]

     [project.urls]
     Repository = "https://github.com/username/my-mlops-project"
     Documentation = "https://username.github.io/my-mlops-project"

     # PEP 735 dependency groups (not shipped with the package).
     [dependency-groups]
     dev = [
       "lefthook>=2.1.10",
       "pip-audit>=2.10.1",
       "pytest>=9.1.1",
       # Ruff 0.16 rewrote the default rule set and now formats Python inside Markdown:
       # an older Ruff disagrees with a 0.16-formatted repository, so floor it here.
       "ruff>=0.16.2",
       "ty>=0.0.69,<0.1", # pre-1.0: pin a compatible range
     ]

     [build-system]
     # Keep the upper bound at least one minor ahead of the pinned `uv`: without it
     # `uv build` warns, and a future breaking `uv_build` silently breaks the sdist.
     requires = ["uv_build>=0.9,<0.13"]
     build-backend = "uv_build"
     ```

### 3. Dependency Management

Establish a clean separation between production and development dependencies.

1. **Add Runtime Dependencies** (Production):
   - Use `uv add <package>` for libraries needed in production (e.g., `mlflow`, `pandas`, `scikit-learn`).
   - These go into `[project.dependencies]` in `pyproject.toml`.
1. **Add Dev Dependencies** (Development):
   - Use `uv add --dev <package>` (or `--group dev`) for tools like `pytest`, `ruff`, `ty`.
   - These go into `[dependency-groups]` (PEP 735) and are kept out of production builds.
1. **Sync Environment**:
   - Run `uv sync` to resolve dependencies, create the `.venv`, and generate the `uv.lock` file.
   - **Critical**: The `uv.lock` file pins exact versions of all dependencies (including transitive ones). It ensures that every developer and CI/CD pipeline uses the exact same environment, preventing "it works on my machine" issues. Commit this file to git.

### 4. Version Control (Git)

Set up a clean repository and ensure unwanted files are ignored.

1. **Initialize Git**:
   - `git init`
   - `git branch -M main`
1. **Create `.gitignore`**:
   - Write a robust `.gitignore` tailored for Python/MLOps.
   - **Must Include**:
     - Environment: `.venv/`, `.env`
     - Caches: `__pycache__/`, `.pytest_cache/`, `.ruff_cache/`, `.ty_cache/`
     - Builds: `dist/`, `build/`, `*.egg-info/`
     - Data/Models: `data/`, `models/`, `outputs/` (unless using DVC/LFS)
     - MLflow local state: `mlflow.db`, `mlartifacts/`, `mlruns/`
     - IDE: `.vscode/` (selectively), `.idea/`, `.DS_Store`
   - _Note_: It is often good practice to commit project-specific `.vscode/settings.json` but ignore `User` settings.
1. **Verify Status**:
   - `git status` should show only source files, config files, and the lockfile.

### 5. Task Runner & Project Instructions

Give every contributor — human or agent — one entrypoint and one written contract.

1. **Task Vocabulary**: Create `mise.toml` with the canonical tasks `install`, `format`, `check`, `test`, `build`, plus `all` = `format` -> `check` -> `test` -> `build`. `mise run all` is the gate: git hooks and CI run that exact task and nothing else, so a green local run means a green pipeline. The [mlops-automation](../mlops-automation/SKILL.md) skill details the tasks.
1. **Pinned Toolchain**: Declare non-Python tools in `[tools]` and run `mise lock`, then commit `mise.lock`. Python dependencies are pinned by `uv.lock`; everything else is pinned by `mise.lock`.
1. **`README.md`** (humans): what the project is, how to install it, how to run it.
1. **`AGENTS.md`** (AI agents): the project overview, the exact commands, the definition of done, the conventions, and the repository layout. Keep it short and current — a stale `AGENTS.md` misleads every assistant that reads it.

### 6. IDE Configuration (VS Code)

Standardize the developer experience (DX) by committing project-specific settings.

1. **Install Recommended Extensions**:
   - **Python Tier A**: `ms-python.python`, `charliermarsh.ruff`, `astral-sh.ty`, `ms-toolsai.jupyter`.
   - **Productivity**: `eamodio.gitlens`, `alefragnani.project-manager`, `usernamehw.errorlens`.
   - **One checker only**: the project type-checks with `ty`, so install Astral's `astral-sh.ty` extension rather than Pylance. It ships the ty language server and sets `python.languageServer` to `"None"` itself, which prevents two checkers from reporting contradictory diagnostics on the same file.
1. **Create `.vscode` Directory**:
   - `mkdir .vscode`
1. **Create `settings.json`**:
   - Configure settings to enforce code quality and use the `uv` environment.
   - **Key Settings**:

     ```json
     {
       "[python]": {
         "editor.defaultFormatter": "charliermarsh.ruff",
         "editor.formatOnSave": true,
         "editor.codeActionsOnSave": {
           "source.organizeImports": "explicit"
         }
       },
       "python.defaultInterpreterPath": ".venv/bin/python",
       "python.terminal.activateEnvironment": true,
       "python.testing.pytestEnabled": true,
       "files.trimTrailingWhitespace": true,
       "files.insertFinalNewline": true,
       "editor.rulers": [120],
       "files.exclude": {
         "**/__pycache__": true,
         "**/.pytest_cache": true,
         "**/.ruff_cache": true,
         "**/.ty_cache": true,
         "**/.venv": true
       }
     }
     ```

   - **Keep the ruler and the linter in agreement**: `editor.rulers` must equal `[tool.ruff] line-length` (120 in this stack). A ruler at 88 against a 120-character limit makes the editor flag code that `ruff` accepts, and developers reformat by hand to satisfy a line that no check enforces.

### 7. Verification & First Commit

Finalize the initialization.

1. **Verify Environment**:
   - Run `uv run python -c "import sys; print(sys.executable)"` to confirm it uses the `.venv`.
1. **Verify the Gate**:
   - Run `mise run all` and fix whatever it reports before the first commit.
1. **Initial Commit**:
   - `git add .`
   - `git commit -m "chore: initialize project with uv, git, mise, and vscode settings"`

### 8. Best Practices Summary

- **One Command Setup**: ideally, `mise run install` (which wraps `uv sync`) should be the only command needed to set up the environment.
- **One Command Gate**: `mise run all` is the single answer to "is this ready?" — locally, in hooks, and in CI.
- **Lockfiles**: Commit both `uv.lock` (Python) and `mise.lock` (tools) so all environments are identical.
- **Editor Config**: Checked-in `.vscode/settings.json` reduces onboarding friction and enforces standards (formatting, linting).
- **Dependency Separation**: Keep production dependencies light; put testing/linting tools in `dev`.

## Self-Correction Checklist

- [ ] **Lockfiles**: Do `uv.lock` and `mise.lock` exist and are they committed?
- [ ] **Virtual Env**: Is `.venv/` created and **ignored** in `.gitignore`?
- [ ] **Project Config**: Does `pyproject.toml` validly describe the project, with an SPDX `license` and `license-files`?
- [ ] **Git Cleanliness**: Are secrets, large data files, and local MLflow state excluded?
- [ ] **Instructions**: Do `README.md` (humans) and `AGENTS.md` (agents) exist and match the real commands?
- [ ] **Gate**: Does `mise run all` pass on a fresh clone?
- [ ] **Reproducibility**: Can another developer `git clone` and `uv sync` to get the exact same state?
