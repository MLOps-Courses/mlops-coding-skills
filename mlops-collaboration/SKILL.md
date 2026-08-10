---
name: mlops-collaboration
description: Prepare a project for public collaboration — license, code of conduct, docs, branch rulesets, templates, and git-cliff releases. Use when open-sourcing a repository, onboarding contributors, or cutting a tagged release.
license: MIT
metadata:
  author: Médéric HURIER (Fmind)
  source: github.com/MLOps-Courses/mlops-coding-skills/tree/main/mlops-collaboration
  created: 2026-01-25
  updated: 2026-08-10
---

# MLOps Collaboration

## Goal

To transform a private project into a public, collaborative resource by establishing **Governance** (License, Code of Conduct, branch rulesets), **Documentation** (README, AGENTS, Contributing), **Standardization** (Templates, Workstations), and **Release Management**.

## Prerequisites

- **Language**: Python 3.14
- **Platform**: GitHub
- **Context**: Open sourcing or team collaboration.

## Instructions

### 1. Repository Governance

Set the rules of engagement.

1. **License**: Pick an SPDX identifier and commit the full text. Declare it in `pyproject.toml` as `license = "MIT"` plus `license-files = ["LICENSE.txt"]` (PEP 639), and make sure the file itself carries its title and copyright line — a bare license body with no `Copyright (c) <year> <author>` is legally ambiguous.
1. **Code of Conduct**: Add `CODE_OF_CONDUCT.md` to foster a safe community.
1. **Branch Protection (concretely)**: commit `.github/rulesets/main.json` and apply it with `mise run install:rulesets`. A ruleset in the repository is reviewable, diffable, and restorable; a setting clicked in the web UI is none of those. A useful baseline blocks deletion and non-fast-forward pushes, requires linear history, requires a pull request, and requires the CI status check to pass.
   - Make the task **idempotent**: look the ruleset up by name and `PUT` over it when it exists, `POST` only when it does not. A plain `POST` creates a duplicate ruleset on every run.
   - The required status check must name the CI **job id** exactly. If you rename the job, the ruleset waits forever for a context that no longer reports.
1. **Review**: Automate preliminary reviews with tools like **Gemini Code Assist** (`.gemini/config.yaml`).
1. **Ignore**: Comprehensive `.gitignore` (exclude secrets, data, virtualenvs, and local MLflow state such as `mlflow.db` and `mlartifacts/`).

### 2. Comprehensive Documentation

Make the project usable and understandable.

1. **README.md**: The landing page for humans (Badges, Hook, Quickstart, commands).
1. **AGENTS.md**: The landing page for AI assistants — project overview, setup and core commands, definition of done, conventions and idioms, repository layout, in that order. Keep both files in sync with reality; when a command changes, both change in the same commit.
1. **Describe the real stack**: state the versions a newcomer will actually install — Python 3.14, MLflow 3.15 on a SQL tracking store (`sqlite:///mlflow.db` locally, not the deprecated file store), Ruff 0.16, `ty` 0.0.69, `uv`, `mise`. A README that documents last year's stack costs more time than no README.
1. **MkDocs**: Use for full documentation sites (API ref, tutorials) when `README.md` gets too long.
1. **CONTRIBUTING.md**: Guide for developers — environment setup, branch naming, PR process, and the exact local gate (`mise run all`) they must pass before opening a pull request.
1. **CHANGELOG.md**: Generate from **Conventional Commits** with `git-cliff` (replaces Commitizen); commit the rendered file.

### 3. Standardization & Workstations

Eliminate "it works on my machine".

1. **Templates**: Use `cookiecutter` for scaffolding and `cruft update` to keep projects synced.
1. **Baseline (required)**: `mise` pins the toolchain (`mise.lock`) and `uv` pins the Python dependencies (`uv.lock`). Together they are what actually makes two machines identical, and they work with any editor.
1. **Devcontainer (recommended, not required)**: `.devcontainer/devcontainer.json` adds one-click GitHub Codespaces and a pinned OS-level image. It is a genuine improvement for teams onboarding non-Python contributors, but it is **not** part of the course's reference package or its cookiecutter template today — so treat it as an upgrade to propose, not a box a project must tick. If you add one, install `mise` in the image and let it install everything else, so the devcontainer and a bare laptop resolve the same versions.

### 4. Release Management

Ship with confidence.

1. **Versioning**: Follow **SemVer** (MAJOR.MINOR.PATCH) driven by **Conventional Commits**.
1. **Changelog**: Generate with `git-cliff` from the commit history (replaces Commitizen/Keep-a-Changelog by hand).
1. **Workflows**:
   - **GitHub Flow**: Small teams, continuous delivery (`main` is stable).
   - **Git Flow**: Scheduled releases (`develop` + `release` branches).
   - **Forking**: Open source, distributed contributors.
1. **Process**: `mise run all` green -> bump version -> `git-cliff` changelog -> annotated `vX.Y.Z` tag (`git tag -a vX.Y.Z`) -> `gh release create vX.Y.Z`.

## Self-Correction Checklist

- [ ] **License**: Is a `LICENSE` file present, titled, copyrighted, and declared with SPDX + `license-files`?
- [ ] **Readme**: Does `README.md` have installation instructions that match the real commands?
- [ ] **Agents**: Does `AGENTS.md` exist and describe the current stack and gate?
- [ ] **Contributing**: Does `CONTRIBUTING.md` state the branch convention and require `mise run all`?
- [ ] **Protection**: Is `.github/rulesets/main.json` committed and applied idempotently by `mise run install:rulesets`?
- [ ] **Reproducibility**: Are `uv.lock` and `mise.lock` committed? (A devcontainer is a recommended bonus, not a requirement.)
- [ ] **SemVer**: Are releases semver-tagged (`vX.Y.Z`) via `gh release create`?
- [ ] **Changelog**: Is `CHANGELOG.md` generated by `git-cliff` from Conventional Commits?
