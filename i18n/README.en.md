# Cobbler Skill

> Make AI your Skill research assistant.

**Works with all major Coding Agents (OpenClaw / Codex / Claude Code / Cursor) without locking into one agent.**

<p align="center">
  <a href="../README.md"><img alt="中文" src="https://img.shields.io/badge/%E8%AF%AD%E8%A8%80-%E4%B8%AD%E6%96%87-blue"></a>
  <a href="./README.en.md"><img alt="English" src="https://img.shields.io/badge/Language-English-lightgrey"></a>
  <a href="../LICENSE"><img alt="License" src="https://img.shields.io/badge/License-MIT-orange"></a>
</p>

## What It Is

Cobbler is a Skill for researching and optimizing local Skill/MCP/CLI tools. It helps you:

- Parse GitHub/GitLab/Gitee repositories and understand their tool ecosystem and framework design.
- Audit local Python environments and manage libraries without recreating environments unnecessarily.
- Research whether local tools fit a target project.
- Diagnose and fix adaptation issues when an agent calls local tools.
- Identify and supplement local capabilities missing from large language models.

## Five Frameworks

### Framework 1: Parse GitHub Repositories

Pass in a GitHub URL, then produce a structured analysis report:

- Project structure, entry files, and core modules.
- Agent framework type, such as Plan-and-Execute, MRKL, ReAct, or Toolformer.
- Skill / MCP / CLI configuration patterns.
- Dependencies and version requirements.

Triggers include:

- "Research this repo"
- "Analyze this GitHub project"
- "What framework is this repo?"
- "parse github"
- "Analyze repository"

### Framework 2: Python Environment Audit and Library Management

Audit local Python environments to avoid duplicate libraries or conflicting virtual environments.

- List all available Python interpreter paths and versions.
- List key installed libraries for each interpreter.
- Check whether a target library is already installed.
- Ask for user confirmation before installing new libraries.
- Prefer reusing existing environments instead of creating new ones.

### Framework 3: Local Framework Fit Research

Given a GitHub repository and an existing local implementation, research whether the local framework fits the target project.

- Does the local framework match the target repository's design intent?
- Are there capability gaps or conflicts?
- Are versions compatible?
- Return a clear result: fits / needs changes / does not fit.

### Framework 4: Agent Adaptation Bug Research

Diagnose issues when Codex, Claude Code, OpenClaw, or another agent calls local tools.

- Skill issues: `SKILL.md` format, trigger clarity, path problems.
- MCP issues: registration failures, protocol mismatches, environment variables.
- CLI issues: command not found, bad arguments, Windows path problems.
- Root cause analysis with concrete repair steps.

### Framework 5: Capability Supplement Research

Research local tool capabilities that a model does not have natively.

- Recommend needed Skill configuration.
- Generate bridge scripts or adapter layer code.
- Propose MCP server supplement configuration.
- Produce concrete capability extension steps.

## Output Contract

Every framework output should include:

1. Diagnosis: current state and problem.
2. Research plan: steps and tools.
3. Concrete repair plan: exact files, lines, or commands.
4. Verification commands.
5. Conflict and compatibility notes.
6. Remaining risk.

## Directory Structure

```text
cobbler-skill/
├── SKILL.md
├── README.md
├── i18n/
│   └── README.en.md
├── LICENSE
├── references/
│   ├── repo-analysis-checklist.md
│   ├── local-plugin-coupling.md
│   ├── mcp-cli-debug-checklist.md
│   ├── skill-maintenance-checklist.md
│   └── python-env-checklist.md
└── agents/
    └── evaluator.md
```

## Scope

Cobbler Skill does:

- GitHub/GitLab/Gitee repository parsing and framework analysis.
- Python/Conda environment audit and library management.
- Local tool fit evaluation.
- Bug diagnosis and repair planning.
- Capability gap identification and supplement suggestions.

Cobbler Skill does not:

- Directly modify user files without confirmation.
- Perform destructive operations such as deleting environments or uninstalling packages.
- Access unauthorized repositories or APIs.
- Compile generated code for the user.

## Reference

This Skill is inspired by the repository analysis workflow in [repo2skill](https://github.com/zhangyanxs/repo2skill), but has a different focus. `repo2skill` focuses on converting repositories into Skills; Cobbler focuses on cross-repository agent framework research and local adaptation.

## License

MIT License
