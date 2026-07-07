# MLOps Coding Skills

Welcome to the **MLOps Coding Skills** repository! 🚀

This project contains a collection of **[Agent Skills](https://agentskills.io/home)** designed to accompany the [MLOps Coding Course](https://mlops-coding-course.fmind.dev/). These skills are tailored to help AI coding assistants (like Antigravity, Gemini CLI, GitHub Copilot, and Claude Code) understand and guide you through the best practices of MLOps, from prototyping to production.

## 🧠 What are Agent Skills?

[Agent Skills](https://agentskills.io/home) are specialized context or instructions that you can load into your AI assistant. Each skill is a folder containing a `SKILL.md` file (YAML frontmatter + Markdown instructions) that provides the "Know-How" for a specific task, ensuring the AI follows the course's methodology, standards, and patterns.

## 📚 Skills in this repository

Each skill derives from one chapter of the [MLOps Coding Course](https://mlops-coding-course.fmind.dev/):

| Skill                     | Course chapter     | Focus                                                                       |
| ------------------------- | ------------------ | --------------------------------------------------------------------------- |
| `mlops-initialization`    | 1. Initializing    | Set up a robust project structure with `uv`, `git`, and VS Code.            |
| `mlops-prototyping`       | 2. Prototyping     | Write clean, reproducible Jupyter notebooks and prevent data leakage.       |
| `mlops-industrialization` | 3. Productionizing | Turn prototypes into a distributable package (src layout, hybrid paradigm). |
| `mlops-validation`        | 4. Validating      | Add typing (`ty`), linting (`ruff`), testing (`pytest`), and logging.       |
| `mlops-automation`        | 5. Refining        | Automate tasks, containers, CI/CD, and experiment tracking (MLflow).        |
| `mlops-collaboration`     | 6. Sharing         | Prepare the project for sharing: license, docs, templates, releases.        |
| `mlops-observability`     | 7. Observability   | Add reproducibility, lineage, monitoring, alerting, and explainability.     |

## 🛠️ Installation & Usage

Skills follow the open `SKILL.md` standard, so most agents discover them from a well-known folder. The workspace-standard location is `.agents/skills/`; each tool then reads from its own path (see the table below).

### Recommended setup (symlinks)

1. **Clone this repository** to a location on your machine (e.g. `~/mlops-coding-skills`).
1. **Create the skills directory** in your target project or workspace, then **symlink** the skills you need:

   ```bash
   mkdir -p .agents/skills
   # Symlink every MLOps skill (or pick individual folders):
   ln -s ~/mlops-coding-skills/mlops-* .agents/skills/
   ```

1. **Expose them to your agent** using the per-tool path below.

### Per-tool install paths

| Tool                      | Skills directory                                     | Notes                                                                             |
| ------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------- |
| **Antigravity**           | `.agents/skills/` (workspace)                        | Discovered natively; no extra step once symlinked.                                |
| **Gemini CLI**            | `.gemini/skills/` (workspace) or `~/.gemini/skills/` | Install via `gemini skill install <git-url> --scope=workspace`.                   |
| **Claude Code**           | `.claude/skills/` (project) or `~/.claude/skills/`   | Project skills live in `.claude/skills/`; personal skills in `~/.claude/skills/`. |
| **Claude Code (plugins)** | `<plugin>/skills/`                                   | Skills bundled in a plugin are loaded automatically when the plugin is installed. |
| **GitHub Copilot**        | any workspace path                                   | Open the relevant `SKILL.md` to load it into context (Agent Mode).                |

### Antigravity

**Antigravity** natively supports the `.agents/skills` standard. Once the symlinks from the [Recommended setup](#recommended-setup-symlinks) are in place, Antigravity detects and uses these skills automatically.

_See the [Antigravity Skills docs](https://antigravity.google/docs/skills) for details._

### Gemini CLI

The **Gemini CLI** manages skills in `.gemini/skills`. Install this repository as a remote skill set to keep it updated and scoped correctly:

```bash
gemini skill install https://github.com/MLOps-Courses/mlops-coding-skills --scope=workspace
```

_See the [Gemini CLI Skills docs](https://geminicli.com/docs/cli/skills/) for details._

### Claude Code

**Claude Code** discovers skills natively — no manual context loading required:

- **Personal skills** (all your projects): place or symlink the folders under `~/.claude/skills/`, e.g. `ln -s ~/mlops-coding-skills/mlops-* ~/.claude/skills/`.
- **Project skills** (this workspace only): Claude Code reads workspace skills from `.claude/skills/`. If you keep them under the standard `.agents/skills/`, link it once: `mkdir -p .claude && ln -s ../.agents/skills .claude/skills`.
- **Plugin skills**: any skill bundled inside an installed plugin's `skills/` directory is loaded automatically.

Claude Code then loads a skill on demand whenever its `description` matches your task.

_(For **Claude Desktop**, add the relevant `SKILL.md` files to a Project's Knowledge instead.)_

_See the [Claude Skills documentation](https://code.claude.com/docs/en/skills) for details._

### GitHub Copilot

For **GitHub Copilot** (Agent Mode, e.g. in VS Code), reference the skill files explicitly:

1. Open your project in VS Code.
1. Open the relevant `SKILL.md` file (e.g. `.agents/skills/mlops-observability/SKILL.md`) to load it into context.
1. Ask Copilot to follow the instructions in the open skill file.

_See [GitHub Copilot Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) for the latest features._

## 🤝 Contributing

We welcome contributions! If you have a new MLOps pattern or improvement, please open a Pull Request. See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`AGENTS.md`](AGENTS.md) for the repository conventions and local checks (`mise run format`, `mise run check`).

Happy Coding! 🤖✨
