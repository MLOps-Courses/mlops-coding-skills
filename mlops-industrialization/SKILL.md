---
name: mlops-industrialization
description: Convert notebook prototypes into a distributable Python package with a src layout, a domain/io/application split, and validated OmegaConf plus Pydantic configuration. Use when moving code out of notebooks or designing entrypoints.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-industrialization
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Industrialization

## Goal

To convert experimental code (notebooks/scripts) into a high-quality, distributable Python package. This skill enforces the **src/ layout**, a **Hybrid Paradigm** (OOP structure + Functional purity), and **Strict Configuration** to ensure scalability, security, and maintainability.

## Prerequisites

- **Language**: Python 3.14
- **Manager**: `uv`
- **Context**: Moving from `notebooks/` to `src/`.

## Instructions

### 1. Packaging Structure (`src` Layout)

Adopt the `src` layout to prevent import errors and separate source from tooling.

1. **Directory Tree**:

   ```text
   my-project/
   ├── pyproject.toml       # Dependencies & Metadata
   ├── uv.lock              # Pinned Python dependencies
   ├── mise.toml            # Task vocabulary & pinned tools
   ├── mise.lock            # Pinned tool binaries
   ├── AGENTS.md            # Instructions for AI agents
   ├── README.md
   └── src/
       └── my_package/      # Main package directory
           ├── __init__.py
           ├── io/          # Side-effects (Datasets, APIs)
           ├── domain/      # Pure business logic (Models, Features)
           └── application/ # Orchestration (Training loops, Inference)
   ```

1. **Configuration**: Use `pyproject.toml` for all build metadata and dependencies.

### 2. Modularity & Paradigm (Hybrid Style)

Balance structure with predictability.

1. **Domain Layer (Pure)**:
   - **Rule**: Code here must be deterministic and free of side effects (no I/O).
   - **Use Case**: Feature transformations, Model architecture definitions.
   - **Style**: Functional (pure functions) or Immutable Objects (dataclasses).
1. **I/O Layer (Impure)**:
   - **Rule**: Isolate external interactions here.
   - **Use Case**: Loading data from S3, saving models to disk, logging to MLflow.
   - **Style**: OOP (Classes to manage connections/state).
1. **Application Layer (Orchestration)**:
   - **Rule**: Wire Domain and I/O together.
   - **Use Case**: Tuning, Training, Inference, Evaluation, etc.

### 3. Application Entrypoints

Create standard, installable CLI tools.

1. **Define Script**: Create `src/my_package/scripts.py` with a `main()` function.
1. **Register**: Add to `pyproject.toml`:

   ```toml
   [project.scripts]
   my-tool = "my_package.scripts:main"
   ```

1. **CLI Execution**:
   - **Dev**: `uv run my-tool` (No install needed).
   - **Prod**: `pip install .` -> `my-tool` (Installed on PATH).
1. **Guard**: Always use `if __name__ == "__main__":` in scripts to prevent execution on import.

### 4. Configuration Management

Decouple settings from code using **OmegaConf** (Parsing) and **Pydantic** (Validation).

1. **Define Schema (Pydantic)**:
   - Create a class that defines _expected_ types and defaults.

   ```python
   from pydantic import BaseModel


   class TrainingConfig(BaseModel):
       batch_size: int = 32
       learning_rate: float = 0.001
       use_gpu: bool = False
   ```

1. **Parse & Validate (OmegaConf)**:
   - Load YAML, merge with CLI args, and validate against the schema.

   ```python
   import omegaconf

   # 1. Load YAML
   conf = omegaconf.OmegaConf.load("config.yaml")
   # 2. Merge with CLI (optional)
   cli_conf = omegaconf.OmegaConf.from_cli()
   merged = omegaconf.OmegaConf.merge(conf, cli_conf)
   # 3. Validate -> Returns a validated Pydantic object
   cfg: TrainingConfig = TrainingConfig(**omegaconf.OmegaConf.to_container(merged))
   ```

1. **Secrets**: Use Environment Variables (`os.getenv`) or `pydantic-settings`, never commit them.

### 5. MLflow Services as I/O Objects

Tracking is a side effect, so it belongs in the I/O layer behind a small, configurable service.

1. **Backend**: Default the tracking and registry URIs to a SQL store — `sqlite:///mlflow.db` locally, a Postgres or HTTP tracking server in production. The file store is deprecated in MLflow 3.15 and does not support the model registry, so it is not a valid default for a package meant to reach production.
1. **Configuration, not constants**: Expose `tracking_uri`, `registry_uri`, `experiment_name`, and `autolog` as validated fields, so the same package runs against a laptop database and a shared server without a code change.
1. **Lifecycle**: Give the service explicit `start()`/`stop()` methods called by the application layer, never at import time.

### 6. Documentation & Quality

Make code usable and maintainable.

1. **Docstrings**: Use **Google Style** docstrings for all modules, classes, and functions.

   ```python
   def calculate_metric(y_true: np.ndarray, y_pred: np.ndarray) -> float:
       """Calculates the accuracy score.

       Args:
           y_true: Ground truth labels.
           y_pred: Predicted labels.

       Returns:
           The accuracy as a float between 0 and 1.
       """
   ```

1. **Type Hints**: Use modern Python typing (`list[str]`, `X | Y`) everywhere; `ty` (0.0.69+) checks them.
1. **Instructions**: Record the layer boundaries, the naming conventions, and the exact commands in `AGENTS.md` so assistants stop guessing where new code belongs.
1. **Gate**: `mise run all` (format -> check -> test -> build) must pass before a refactor is considered finished — see [mlops-validation](../mlops-validation/SKILL.md).

### 7. Best Practices Summary

- **Config != Code**: Never hardcode paths or hyperparams; use the `Pydantic + OmegaConf` pattern.
- **Entrypoints are APIs**: Design your CLI (`[project.scripts]`) as the public interface for your automation tools.
- **Immutable Core**: Keep your domain logic side-effect free; push I/O to the edges.

## Self-Correction Checklist

- [ ] **No Side Effects on Import**: Does `import my_package` run any code? (It shouldn't).
- [ ] **Src Layout**: Is code inside `src/`?
- [ ] **Config Safety**: Are secrets excluded from `pyproject.toml` and YAML?
- [ ] **Typing**: Are function signatures fully type-hinted and does `ty check` pass?
- [ ] **Entrypoints**: Is the CLI registered in `pyproject.toml`?
- [ ] **Tracking**: Does the MLflow service default to a SQL backend rather than the file store?
- [ ] **Gate**: Does `mise run all` pass?
