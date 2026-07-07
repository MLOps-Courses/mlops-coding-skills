# AGENTS.md

Context and rules for AI agents working in this repository. Humans should start with `README.md`.

## Project overview

- **Name**: mlops-coding-skills — a collection of Anthropic-style [Agent Skills](https://agentskills.io/home) for MLOps.
- **Purpose**: package the methodology of the [MLOps Coding Course](https://mlops-coding-course.fmind.dev) as reusable skills that AI coding assistants (Antigravity, Gemini CLI, GitHub Copilot, Claude Code) can load on demand.
- **Content**: each `mlops-*/` folder holds one `SKILL.md` (YAML frontmatter + Markdown instructions). There is no application code to build.

## Repository layout

- `mlops-<topic>/SKILL.md` — one skill per course chapter: `initialization`, `prototyping`, `industrialization`, `validation`, `automation`, `collaboration`, `observability`.
- `mise.toml` — task vocabulary and pinned tools; `lefthook.yml` — git hooks; `dprint.jsonc` — formatter config; `cliff.toml` — changelog config; `.github/workflows/ci.yml` — CI.
- `README.md` (humans) and `AGENTS.md` (agents); `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `LICENSE` — governance.

## Setup & core commands

All work goes through `mise` (see `mise.toml`); git hooks (`lefthook.yml`) and CI call the same tasks.

- Install: `mise run install` — install git hooks (`lefthook`).
- Format: `mise run format` — `dprint` formats JSON, Markdown, TOML, YAML.
- Check: `mise run check` — `dprint check` (format), `gitleaks` (secrets in git history), and `check:frontmatter` (each `SKILL.md` frontmatter `name` is slug-cased and equals its folder name).

## Conventions & idioms

- **SKILL.md frontmatter**: `name` must match `^[a-z0-9-]+$` and equal the folder name (e.g. `mlops-validation`); `description` is a single line stating when to use the skill. The `check:frontmatter` task enforces this — fix names, never weaken the check.
- **One skill, one chapter**: keep each `SKILL.md` scoped to a single course chapter; cross-reference rather than duplicate.
- **Formatting**: `dprint` is authoritative for config and Markdown; do not hand-wrap Markdown (`textWrap: never`).
- **Commits**: Conventional Commits (`feat:`, `fix:`, `refactor:`, `chore:`); no attribution in commit messages. Releases use `git-cliff` (`cliff.toml`) with `vX.Y.Z` tags.

## Definition of done

A change is complete only when, locally, `mise run format` is clean and `mise run check` reports no findings. Fix root causes — never weaken an assertion or skip a check to force a green result.
