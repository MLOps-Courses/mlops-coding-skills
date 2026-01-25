# MLOps Coding Skills

Welcome to the **MLOps Coding Skills** repository! 🚀

This project contains a collection of **[Agent Skills](https://agentskills.io/home)** designed to accompany the [MLOps Coding Course](https://mlops-coding-course.fmind.dev/). These skills are tailored to help AI coding assistants (like Gemini, Copilot, and Claude) understand and guide you through the best practices of MLOps, from prototyping to production.

## 🧠 What are Agent Skills?

[Agent Skills](https://agentskills.io/home) are specialized context or instructions that you can load into your AI assistant. They provide the "Know-How" for specific tasks, ensuring that the AI follows the course's methodology, standards, and patterns.

This repository covers skills for:

- **Initialization**: Setting up robust project structures.
- **Prototyping**: Writing clean, experimental code.
- **Industrialization**: Optimizing for production deployment.
- **Validation**: Testing data, models, and code.
- **Automation**: CI/CD and workflow automation.
- **Collaboration**: Working effectively in teams.
- **Observability**: Implementing logging, lineage, and monitoring.

## 📂 Repository Structure

The skills are organized by topic. Each top-level folder corresponds to a domain of MLOps.

```bash
.
├── mlops-initialization/  # Initialization skills
├── mlops-prototyping/     # Prototyping skills
├── mlops-industrialization/ # Industrialization skills
├── mlops-validation/      # Validation skills
├── mlops-automation/      # Automation skills
├── mlops-collaboration/   # Collaboration skills
├── mlops-observability/   # Observability skills
└── README.md
```

## 🛠️ Installation & Usage

### Recommended Setup (Symlinks)

To use these skills alongside others, we recommend creating a `.agent/skills` folder in your project directory and creating symlinks to the specific skills you need from this repository.

1. **Clone this repository** to a location on your machine (e.g., `~/mlops-coding-skills`).
2. **Create a skills directory** in your target project or workspace:

    ```bash
    mkdir -p .agent/skills
    ```

3. **Symlink the skills** you want to use. For example, to use all MLOps skills:

    ```bash
    # Example: Symlink all skill folders
    ln -s ~/mlops-coding-skills/mlops-* .agent/skills/
    ```

### 1. Antigravity

**Antigravity** natively supports the `.agent/skills` standard. If you have set up the symlinks as described in the [Recommended Setup](#recommended-setup-symlinks) section, Antigravity will automatically detect and use these skills.

*See [Antigravity Skills Docs](https://antigravity.google/docs/skills) for more details.*

### 2. Gemini CLI

The **Gemini CLI** manages skills in `.gemini/skills`. We recommend installing this repository as a remote skill set to keep it updated and scoped correctly.

**Install via Git (Recommended)**
This command installs the skills into your current workspace scope.

```bash
gemini skill install https://github.com/MLOps-Courses/mlops-coding-skills --scope=workspace
```

*See [Gemini CLI Skills Docs](https://geminicli.com/docs/cli/skills/) for more details.*

### 3. GitHub Copilot

For **GitHub Copilot**, you can provide these skills as context in your workspace.

**Agent Mode:**
If you are using Copilot in Agent Mode (e.g., in VS Code), you can explicitly reference the skill files when asking for help with a specific MLOps task.

1. Open your project in VS Code.
2. Open the relevant `SKILL.md` file (e.g., `.agent/skills/mlops-observability/SKILL.md`) to load it into context.
3. Ask Copilot: "Follow the instructions in the open skill file to help me implement observability."

*See [GitHub Copilot Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) for the latest features.*

### 4. Claude Desktop / Claude Code

For **Claude**, you can use these skills as "Project Knowledge" or explicit context.

**Claude Desktop:**

1. Create a "Project" in Claude.
2. Add the relevant `SKILL.md` files from this repository (or your `.agent/skills` folder) to the Project Knowledge.
3. Claude will now follow these guidelines when answering your queries within that project.

*See [Claude Skills Documentation](https://code.claude.com/docs/en/skills) for more advanced integrations.*

## 🤝 Contributing

We welcome contributions! If you have a new MLOps pattern or improvement, please open a Pull Request.

Happy Coding! 🤖✨
