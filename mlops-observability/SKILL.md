---
name: mlops-observability
description: Make an ML system a glass box with reproducible runs, MLflow dataset lineage, drift monitoring, alerting, and SHAP explanations. Use when a deployed model needs traceability, monitoring, alerting, or explanation.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-observability
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Observability

## Goal

To implement a "Glass Box" system where every result is **Reproducible**, every asset has **Lineage**, and system health is **Monitored**, **Alerted** on, and **Explained**.

## Prerequisites

- **Language**: Python 3.14
- **Context**: Production monitoring and debugging.
- **Platform Suggestion**: MLflow 3.15, SHAP, Evidently, ...

## Instructions

### 1. Guarantee Reproducibility

Consistency is key. For instance:

1. **Randomness**: Set seeds for `random`, `numpy`, `torch`, `tensorflow`.
1. **Dependencies**: `uv.lock` is the reproducibility mechanism for Python. It records the exact resolved version and hash of every direct and transitive dependency, and `uv sync --frozen` installs exactly that — the same set on a laptop, in CI, and in the image.
1. **Tools**: `mise.lock` does the same job for the binaries that are not Python packages (`dprint`, `gitleaks`, `trivy`, `actionlint`, `zizmor`, ...), recording version, URL, and checksum per platform. Commit both lockfiles; between them, "works on my machine" stops being a category of bug.
1. **Builds**: `mise run build` is a plain `uv build` producing a wheel and an sdist. Its reproducibility comes from the locked inputs above, not from a build flag — do not expect `uv build` to pin anything by itself.
1. **Environment**: Ship the same locked set into a `docker` image (`uv sync --frozen`), so the runtime matches what was tested.
1. **Code**: Track the git commit hash for every run, and fail the pipeline on a dirty working tree so a run can always be traced back to a commit.

### 2. Track Data Lineage

Know the origin of your data. For instance:

1. **Datasets**: Create MLflow Datasets with `mlflow.data.from_pandas`.
1. **Logging**: Log inputs to MLflow context with `mlflow.log_input`.
1. **Store**: Keep tracking and registry on a SQL backend (`sqlite:///mlflow.db` locally, Postgres or a tracking server in production). Lineage queries are relational queries; the deprecated file store cannot answer them and does not support the model registry at all.
1. **Versioning**: Version data files (e.g., `data/v1.csv`) or use DVC.
1. **Transformations**: Log preprocessing parameters mapping data versions to model versions.

### 3. Monitoring & Drift Detection

Watch for silent failures. For instance:

1. **Validation**: Gate models against quality thresholds with `mlflow.validate_evaluation_results` (MLflow 3).
1. **Drift**: Use `evidently` to compare `reference` (training) vs `current` (production) data.
   - Detect Data Drift (input distribution changes) and Concept Drift (relationship changes).
1. **System**: Enable MLflow System Metrics (`log_system_metrics=True`) for CPU/GPU.

### 4. Alerting

Don't stare at dashboards. For instance:

1. **Local**: Use `plyer` for desktop notifications during long training runs.
1. **Production**: Use `PagerDuty` (critical) or `Slack` (warnings).
1. **Thresholds**: Use Static (fixed value) or Dynamic (anomaly detection) rules.
1. **Action**: Alerts must link to a dashboard or playbook.

### 5. Explainability (XAI)

Trust but verify. For instance:

1. **Global**: Use Feature Importance (e.g., Random Forest) to understand overall logic.
1. **Local**: Use `SHAP` values to explain _individual_ predictions.
1. **Artifacts**: Save explanations (plots/tables) as MLflow artifacts.

### 6. Infrastructure & Costs

Optimize resources. For instance:

1. **Tags**: Tag runs with `project`, `env`, `user`.
1. **Costs**: Log `run_time` and instance type to estimate ROI.

### 7. Observability of the Repository Itself

The pipeline that produces the model needs the same treatment.

1. **One Gate**: `mise run all` (format -> check -> test -> build) is the signal that a change is releasable. CI runs that exact task, so a green pipeline and a green laptop mean the same thing.
1. **Static Guarantees**: Ruff 0.16 and `ty` 0.0.69 run inside `mise run check`, alongside the `pip-audit`, `gitleaks`, and `trivy` scans — quality signals you get on every commit, not once a quarter.
1. **Written Down**: `AGENTS.md` records the commands, the definition of done, and the conventions, so an AI assistant debugging a production incident reads the same runbook a human does.

## Self-Correction Checklist

- [ ] **Seeds**: Are random seeds fixed?
- [ ] **Lockfiles**: Are `uv.lock` and `mise.lock` committed, and does the image install with `--frozen`?
- [ ] **Inputs**: Are input datasets logged to MLflow, on a SQL-backed store?
- [ ] **System Metrics**: Is `log_system_metrics` enabled?
- [ ] **Explanations**: Are SHAP values generated and stored as artifacts?
- [ ] **Alerts**: Are thresholds defined for failures?
- [ ] **Gate**: Does `mise run all` pass, and does CI run that same task?
