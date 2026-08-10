# AGENTS.md

Context and rules for AI agents working in this repository. Humans should start with `README.md`.

## Project overview

- **Name**: mlops-coding-skills — a collection of Anthropic-style [Agent Skills](https://agentskills.io/home) for MLOps.
- **Purpose**: package the methodology of the [MLOps Coding Course](https://mlops-coding-course.fmind.dev) as reusable skills that AI coding assistants (Antigravity, Gemini CLI, GitHub Copilot, Claude Code) can load on demand.
- **Content**: each `mlops-*/` folder holds one `SKILL.md` (YAML frontmatter + Markdown instructions). There is no application code to build and no test suite to run.
- **Reference**: the skills describe the stack implemented in [mlops-python-package](https://github.com/fmind/mlops-python-package) — Python 3.14, `uv`, `mise`, Ruff 0.16, `ty` 0.0.69, `pytest`, MLflow 3.15 on a SQL tracking store (`sqlite:///mlflow.db`, not the deprecated file store). Verify an instruction against that package before writing it.

## Setup & core commands

All work goes through `mise` (see `mise.toml`); git hooks (`lefthook.yml`) and CI call the same tasks.

- Everything: `mise run all` — format, then check. This is the gate; CI runs this exact task and nothing else.
- Install: `mise install` installs the pinned tools; `mise run install` installs the git hooks (`lefthook`).
- Format: `mise run format` — `dprint` formats JSON, Markdown, TOML, and YAML, including the seven `SKILL.md` deliverables.
- Check: `mise run check` — `dprint check` (format), `actionlint` + `zizmor` (workflows), `gitleaks` (secrets in the last 100 commits), `trivy` (filesystem scan), `check:frontmatter` (name equals folder), and `gh skill publish --dry-run` (Agent Skills specification).
- Tools are pinned in `mise.toml` and locked in `mise.lock`; run `mise lock` after changing a version.

## Definition of done

A change is complete only when, locally, `mise run all` passes with no findings and the seven `SKILL.md` files still describe commands that exist in the reference package. Fix root causes — never weaken an assertion, exclude a file, or skip a check to force a green result.

## Conventions & idioms

- **SKILL.md frontmatter**: `name` must match `^[a-z0-9-]+$` and equal the folder name (e.g. `mlops-validation`); `description` states the capability then the trigger (`<capability>. Use when <trigger>.`) in 240 characters or fewer, because hosts route on name and description alone; `license: MIT` and a `metadata` block (`author`, `source`, `created`, `updated`) complete it. `check:frontmatter` and `check:skills` enforce this — fix names, never weaken the checks.
- **Two skill checks on purpose**: `check:frontmatter` is the offline floor (no network, no `gh`, names the offending file); `check:skills` is the real publishing gate (`gh skill publish --dry-run .`), which also rejects leaked install metadata but only warns about a missing `license`.
- **One skill, one chapter**: keep each `SKILL.md` scoped to a single course chapter; cross-reference rather than duplicate.
- **Ordered lists**: write every ordered-list marker as `1.` so renumbering is automatic.
- **Formatting**: `dprint` is authoritative for config and Markdown, `SKILL.md` files included; do not hand-wrap Markdown (`textWrap: never`).
- **Commits**: Conventional Commits (`feat:`, `fix:`, `refactor:`, `chore:`); no attribution in commit messages. Releases use `git-cliff` (`cliff.toml`) with `vX.Y.Z` tags.

## Repository layout

- `mlops-<topic>/SKILL.md` — one skill per course chapter: `initialization`, `prototyping`, `industrialization`, `validation`, `automation`, `collaboration`, `observability`.
- `mise.toml` / `mise.lock` — task vocabulary and pinned, locked tools; `lefthook.yml` — git hooks; `dprint.jsonc` — formatter config; `trivy.yaml` — scanner policy; `cliff.toml` — changelog config.
- `.github/` — `workflows/` (`ci.yml` runs `mise run all`, `security.yml` rescans the full history weekly), `dependabot.yml`, `zizmor.yml`, issue and pull-request templates.
- `README.md` (humans) and `AGENTS.md` (agents); `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `LICENSE` — governance.
