# Contributing to MLOps Coding Skills

Thank you for your interest in contributing to the **MLOps Coding Skills** repository! We welcome contributions that help MLOps practitioners and their AI assistants build better systems.

## 🤝 How to Contribute

We accept several types of contributions:

1. **New Skills**: Adding new MLOps patterns or tools.
1. **Improvements**: Enhancing existing skills with better instructions or examples.
1. **Bug Fixes**: Correcting errors in the documentation or logic.

### Process

1. **Fork the repository** to your GitHub account.
1. **Clone your fork** locally, then install the toolchain and git hooks: `mise install && mise run install`.
1. **Create a branch** named `<type>/<slug>`, where `<type>` is a Conventional Commits type (`git switch -c feat/mlops-serving`).
1. **Make your changes**. Each skill lives in its own folder as `mlops-<topic>/SKILL.md`, and the frontmatter `name` must equal the folder name.
1. **Run the gate**: `mise run all` (format, then check). It must pass before you open a pull request — CI runs that exact task and nothing else.
1. **Commit your changes** using [Conventional Commits](https://www.conventionalcommits.org) (`feat:`, `fix:`, `docs:`, `chore:`), so `git-cliff` can generate the changelog.
1. **Push to your branch**.
1. **Open a Pull Request** against the `main` branch of this repository.

## 📝 Style Guide

When writing **Agent Skills** (`SKILL.md`), please follow these guidelines:

- **Frontmatter**: `name` matches `^[a-z0-9-]+$` and equals the folder name; `description` states the capability and the trigger (`<capability>. Use when <trigger>.`) in 240 characters or fewer, because hosts route on the name and description alone; `license: MIT`; and a `metadata` block with `author`, `source`, `created`, and `updated`.
- **Clear Title**: Use a descriptive H1 title, followed by a `## Goal` and a `## Prerequisites` section.
- **Instruction Steps**: Use ordered lists for sequential steps, and write every marker as `1.` so renumbering stays automatic.
- **Self-Correction Checklist**: End with a checklist an agent can verify against a real repository.
- **Code Examples**: Provide copy-pasteable code blocks, each with a language identifier.
- **Markdown**: `dprint` is authoritative; run `mise run format` and never hand-wrap paragraphs (`textWrap: never`).
- **Ground it in reality**: every command, file path, and version in a skill must exist in the reference implementation ([mlops-python-package](https://github.com/fmind/mlops-python-package)). A plausible-sounding instruction that does not work is worse than no instruction.

## 🐛 Reporting Bugs

If you find a bug or have a suggestion, please open an Issue using the provided templates.

Thank you for helping us build the best resource for MLOps Agent Skills!
