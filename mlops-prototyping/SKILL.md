---
name: mlops-prototyping
description: Structure reproducible Jupyter notebooks with a fixed section layout, hoisted configuration, and leakage-free scikit-learn pipelines. Use when exploring a dataset, training a first model, or preparing a notebook for promotion.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-prototyping
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Prototyping

## Goal

To create standardized, reproducible, and production-ready prototypes in Jupyter notebooks. This skill enforces a structured layout (Imports -> Configs -> Load -> EDA -> Modeling -> Eval) and robust engineering practices (Pipelines, Split-Verification) to prevent technical debt and data leakage.

## Prerequisites

- **Language**: Python 3.14
- **Environment**: `uv` managed project (`.venv`), with `ipykernel` in a `notebook` dependency group
- **Context**: Executed within a `.ipynb` file or converting to one.

## Instructions

### 1. Notebook Structure

Enforce the following linear sections in every notebook to ensure readability and maintainability.

1. **Title & Purpose**: H1 Title and a brief description of the experiment goals.
1. **Imports**: Group standard libraries, third-party, and usage-specific imports.
1. **Configs**: Define **Global Constants** (paths, random seeds, hyperparameters) here. No magic numbers deeper in the code.
1. **Datasets**: Load, validate, and split data.
1. **Analysis (EDA)**: Inspect target distributions and correlations.
1. **Modeling**: Define and train `sklearn.pipeline.Pipeline` objects.
1. **Evaluations**: Compute metrics and visualize performance on held-out data.

### 2. Configuration Standards

Expose all "knobs" at the top of the notebook for easy experimentation.

- **Randomness**: Define `RANDOM_STATE = 42` and use it in splits and model initialization.
- **Paths**: Use `pathlib` for robust path handling.

  ```python
  from pathlib import Path

  ROOT = Path("..")
  DATA_PATH = ROOT / "data" / "input.parquet"
  ```

- **Hyperparameters**: Group model params (e.g., `N_ESTIMATORS`, `MAX_DEPTH`).
- **Toggles**: Use booleans for expensive operations (e.g., `USE_GPU = True`, `RUN_GRID_SEARCH = False`).

### 3. Data Management

Ensure data integrity and prevent leakage.

- **Loading**: Prefer `pd.read_parquet` for speed/types, or `pd.read_csv`.
- **Splitting**:
  - **Always** split into `X_train`, `X_test`, `y_train`, `y_test` **before** any data-dependent transformations (imputation, scaling).
  - **Random Split**: Use `sklearn.model_selection.train_test_split` with `stratify` for balanced classification.
  - **Time Series**: Use `sklearn.model_selection.TimeSeriesSplit` if data has a temporal dimension (do NOT shuffle).
  - Use `random_state=RANDOM_STATE`.

### 4. Pipeline Construction

Prohibit raw data transformations on the full dataset.

- **Mandate**: Use `sklearn.pipeline.Pipeline` or `ColumnTransformer`.
- **Why**: Automation of `fit` on train and `transform` on test prevents data leakage.
- **Example**:

  ```python
  from sklearn.compose import ColumnTransformer
  from sklearn.impute import SimpleImputer
  from sklearn.pipeline import Pipeline
  from sklearn.preprocessing import StandardScaler

  CACHE = "./.cache"  # Define a cache directory

  numeric_transformer = Pipeline(
      steps=[
          ("imputer", SimpleImputer(strategy="median")),
          ("scaler", StandardScaler()),
      ]
  )

  preprocessor = ColumnTransformer(
      transformers=[("num", numeric_transformer, numeric_features)]
  )

  # Use 'memory' to cache transformer outputs, speeding up GridSearch
  model = Pipeline(
      steps=[
          ("preprocessor", preprocessor),
          ("classifier", RandomForestClassifier()),
      ],
      memory=CACHE,
  )
  ```

### 5. Experiment Tracking from the Notebook

Prototypes are experiments; record them from the first run rather than retrofitting tracking later.

1. **Backend**: Point MLflow at a SQL store, not the deprecated file store: `mlflow.set_tracking_uri("sqlite:///mlflow.db")`. SQLite is a real SQLAlchemy backend, it supports the model registry, and it is the same store shape as a production Postgres — so moving up later is a URI change, not a rewrite.
1. **Autologging**: `mlflow.autolog()` before `fit` captures parameters, metrics, and the model for scikit-learn without extra code.
1. **Scope**: Keep one MLflow experiment per notebook question, and name runs after the hypothesis being tested.

### 6. Evaluation & Visualization

Go beyond accuracy/MSE.

- **Metrics**: Use `sklearn.metrics` appropriate for the task (F1, ROC-AUC, RMSE, MAE).
- **Baselines**: Compare against a "Dummy" model (mean/mode) to verify learning.
- **Visualization**:
  - **Regression**: Residual plots, Actual vs Predicted.
  - **Classification**: Confusion Matrix, ROC Curve, Precision-Recall.
  - **Feature Importance**: Visualize `feature_importances_` or SHAP values.

### 7. Transition to Production

Facilitate the move from notebook to python package (`src/`).

- **Function Refactoring**: Once a block of code is stable (e.g., a complex data cleaning step), refactor it into a function _within_ the notebook. This makes moving it to a `.py` file trivial later.
- **Cell Tagging**: Use tags like `parameters` (for Papermill) or `export` to mark cells that should be part of the final documentation or automated pipeline.
- **Clean State**: Ensure the notebook runs top-to-bottom (`Restart Kernel and Run All`) without errors before committing.
- **Formatting**: Ruff 0.16 formats Python inside Markdown too, so `mise run format` normalizes the snippets you paste into notes and docs. Run `mise run all` before committing a notebook alongside package code.
- **Next step**: [mlops-industrialization](../mlops-industrialization/SKILL.md) covers the package layout the refactored functions move into.

## Self-Correction Checklist

- [ ] **No Magic Numbers**: Are all parameters in the `Configs` section?
- [ ] **No Data Leakage**: Is `fit` called ONLY on `X_train`?
- [ ] **Reproducibility**: Is `random_state` set for all stochastic operations?
- [ ] **Resilience**: Are paths defined relative to the project root?
- [ ] **Tracking**: Do runs land in a SQL-backed MLflow store rather than the deprecated file store?
- [ ] **Clarity**: Does the notebook read like a report (Markdown cells explaining the _Why_)?
