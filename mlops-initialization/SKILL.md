---
name: mlops-initialization
description: Guide to initialize a new MLOps project with standard tools (uv, git, VS Code) and best practices.
---

# MLOps Initialization

## Goal

To initialize a robust, production-ready MLOps project structure using the modern Python toolchain (`uv`), industry-standard version control (`git`), and a configured development environment (`VS Code`). This skill ensures reproducibility, collaboration, and high code quality from day one.

## Prerequisites

- **Language**: Python 3.14 (latest stable)
- **Manager**: `uv` (replaces pip, venv, poetry, pyenv)
- **VCS**: Git
- **IDE**: VS Code (recommended)

## Instructions

### 1. System & Toolchain Verification

Before modifying files, verify that the essential tools are available.

1. **Check `uv`**:
    - Ensure `uv` is installed: `uv --version`
    - If missing, install it: `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. **Check `git`**:
    - Ensure `git` is installed: `git --version`

### 2. Project Initialization

Initialize the project structure using `uv` to ensure modern standards (`pyproject.toml`).

1. **Create Directory** (if not already inside):
    - `mkdir <project_name> && cd <project_name>`
2. **Initialize Project**:
    - Run `uv init`
    - This creates `pyproject.toml`, `.python-version`, and a basic `hello.py`.
3. **Configure `pyproject.toml`**:
    - Update **metadata**: `name`, `version`, `description`, `authors`, `license`.
    - Set **requires-python**: Ensure it matches the project's target environment (e.g., `>=3.14`).
    - **Example Structure**:

        ```toml
        [project]
        name = "my-mlops-project"
        version = "0.1.0"
        description = "A robust MLOps project."
        readme = "README.md"
        requires-python = ">=3.14"
        license = { file = "LICENSE" }
        authors = [{ name = "Your Name", email = "your.email@example.com" }]
        dependencies = [
            "pandas>=2.3.0",
            "loguru>=0.7.0",
            # Add other runtime dependencies here
        ]

        [project.urls]
        Repository = "https://github.com/username/my-mlops-project"
        Documentation = "https://username.github.io/my-mlops-project"

        # PEP 735 dependency groups (not shipped with the package).
        [dependency-groups]
        dev = [
            "pytest>=9.0.0",
            "ruff>=0.15.0",
            "ty>=0.0.56,<0.1", # pre-1.0: pin a compatible range
        ]

        [build-system]
        requires = ["uv_build>=0.9.0"]
        build-backend = "uv_build"
        ```

### 3. Dependency Management

Establish a clean separation between production and development dependencies.

1. **Add Runtime Dependencies** (Production):
    - Use `uv add <package>` for libraries needed in production (e.g., `fastapi`, `numpy`, `torch`).
    - These go into `[project.dependencies]` in `pyproject.toml`.
2. **Add Dev Dependencies** (Development):
    - Use `uv add --dev <package>` (or `--group dev`) for tools like `pytest`, `ruff`, `ty`.
    - These go into `[dependency-groups]` (PEP 735) and are kept out of production builds.
3. **Sync Environment**:
    - Run `uv sync` to resolve dependencies, create the `.venv`, and generate the `uv.lock` file.
    - **Critical**: The `uv.lock` file pins exact versions of all dependencies (including transitive ones). It ensures that every developer and CI/CD pipeline uses the exact same environment, preventing "it works on my machine" issues. Commit this file to git.

### 4. Version Control (Git)

Set up a clean repository and ensure unwanted files are ignored.

1. **Initialize Git**:
    - `git init`
    - `git branch -M main`
2. **Create `.gitignore`**:
    - Write a robust `.gitignore` tailored for Python/MLOps.
    - **Must Include**:
        - Environment: `.venv/`, `.env`
        - Caches: `__pycache__/`, `.pytest_cache/`, `.ruff_cache/`, `.ty_cache/`
        - Builds: `dist/`, `build/`, `*.egg-info/`
        - Data/Models: `data/`, `models/`, `outputs/` (unless using DVC/LFS)
        - IDE: `.vscode/` (selectively), `.idea/`, `.DS_Store`
        - *Note*: It is often good practice to commit project-specific `.vscode/settings.json` but ignore `User` settings.
3. **Verify Status**:
    - `git status` should show only source files, config files, and the lockfile.

### 5. IDE Configuration (VS Code)

Standardize the developer experience (DX) by committing project-specific settings.

1. **Install Recommended Extensions**:
    - **Python Tier A**: `ms-python.python`, `charliermarsh.ruff`, `ms-python.vscode-pylance`, `ms-toolsai.jupyter`.
    - **Productivity**: `eamodio.gitlens`, `alefragnani.project-manager`, `usernamehw.errorlens`.
2. **Create `.vscode` Directory**:
    - `mkdir .vscode`
3. **Create `settings.json`**:
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
          "python.analysis.typeCheckingMode": "basic",
          "python.testing.pytestEnabled": true,
          "files.trimTrailingWhitespace": true,
          "files.insertFinalNewline": true,
          "editor.rulers": [88],
          "files.exclude": {
            "**/__pycache__": true,
            "**/.pytest_cache": true,
            "**/.ruff_cache": true,
            "**/.venv": true
          }
        }
        ```

### 6. Verification & First Commit

Finalize the initialization.

1. **Verify Environment**:
    - Run `uv run python -c "import sys; print(sys.executable)"` to confirm it uses the `.venv`.
2. **Initial Commit**:
    - `git add .`
    - `git commit -m "chore: initialize project with uv, git, and vscode settings"`

### 7. Best Practices Summary

- **One Command Setup**: ideally, `uv sync` should be the only command needed to set up the environment.
- **Lockfile**: Always commit `uv.lock` to ensure all environments are identical.
- **Editor Config**: Checked-in `.vscode/settings.json` reduces onboarding friction and enforces standards (formatting, linting).
- **Dependency Separation**: Keep production dependencies light; put testing/linting tools in `dev`.

## Self-Correction Checklist

- [ ] **Lockfile**: Does `uv.lock` exist?
- [ ] **Virtual Env**: Is `.venv/` created and **ignored** in `.gitignore`?
- [ ] **Project Config**: Does `pyproject.toml` validly describe the project?
- [ ] **Git Cleanliness**: Are secrets and large data files excluded?
- [ ] **Reproducibility**: Can another developer `git clone` and `uv sync` to get the exact same state?
