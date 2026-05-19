# Cobbler Skill (修鞋匠)

> Your AI Skill Research Assistant

**Works with all major Coding Agents (OpenClaw / Codex / Claude Code / Cursor), no agent lock-in.**

## What It Is

Cobbler helps you research and optimize local Skill/MCP/CLI tools:

- Parse GitHub/GitLab/Gitee repositories to understand tool ecosystem and framework design
- **Python/Conda environment audit and library management** to avoid recreating environments
- Research whether local tools fit the target project
- Diagnose and fix adaptation issues when any Agent calls local tools
- Research and supplement capabilities missing from LLMs

## Five Frameworks

### 🔍 Framework 1: Parse GitHub Repository

Accept a GitHub URL and deeply analyze:

- Project structure, entry points, core modules
- Agent framework type (Plan-and-Execute, MRKL, ReAct, Toolformer, etc.)
- Skill / MCP / CLI configuration patterns
- Dependencies and version requirements

**Triggers** (any of):
- "Research this repo"
- "Analyze this GitHub project"
- "What framework is this repo"
- "Parse github"
- "Analyze repository"

### 🐍 Framework 2: Python Environment Audit & Library Management

Fully audit local Python environments to avoid duplicate library creation or conflicting virtual environments.

- List all available Python interpreters with versions
- List installed core libraries per interpreter
- Check if target library is already installed
- If new installation needed, **require user confirmation** to keep environment clean
- Always reuse existing environments, never recreate

**Triggers** (any of):
- "Check Python environment"
- "Audit local Python environment"
- "Is this library installed"
- "Install xxx"
- "python env"
- "check env"
- "pip list"
- "Show me Python interpreters"
- "Check if torch is installed"
- "What pandas version do I have"

### ⚖️ Framework 3: Research Local Framework Suitability

Accept a GitHub repo + local implementation, research suitability:

- Does the local framework match the target repo's design intent?
- Are there functional conflicts or missing capabilities?
- Is version compatible?
- Clear verdict: ✅ Suitable / ⚠️ Needs modification / ❌ Not suitable

**Triggers** (any of):
- "Check if local framework fits this project"
- "Does my implementation work for this repo's skill"
- "Check local suitability"
- "Framework compatibility"

### 🔧 Framework 4: Research LLM Adaptation Bug

Diagnose issues when Codex / Claude Code / OpenClaw call local tools:

- Skill: SKILL.md format errors, unclear trigger conditions, path issues
- MCP: tool registration failures, protocol mismatches, env var errors
- CLI: command not found, wrong arguments, Windows path issues
- Locate root cause, provide diagnosis and fix plan

**Triggers** (any of):
- "Error occurred"
- "MCP failed to start"
- "CLI command not found"
- "Tool error"
- "Debug this"
- "Diagnose problem"
- "Fix bug"

### 🩹 Framework 5: Research Capability Supplement

Research capabilities missing from local tool ecosystem:

- Recommend needed Skill configurations
- Generate bridging scripts or adapter code
- Propose MCP server supplemental configs
- Generate transformation code blocks

**Triggers** (any of):
- "Missing this capability"
- "What can LLMs not do"
- "Supplement capability"
- "Capability gap"
- "What tools are needed"

## Output Contract

All frameworks output structured reports with these mandatory sections:

```text
1. Diagnosis — What is the current state? What problem exists?
2. Research Plan — What steps will be taken? What tools will be used?
3. Concrete Patches — Exact changes to make (file + line + content)
4. Verification Commands — How to confirm the fix works?
5. Conflict and Coupling Notes — What else is affected?
6. Remaining Risks — What could go wrong?
```

## Quality Standards

- **Specificity**: Each fix includes executable commands or exact file paths
- **Verifiability**: Each fix has a corresponding verification command
- **Honesty**: Unable to execute verification must state reason
- **Completeness**: Report must include conflict and risk sections

## Quick Start

```text
Research this repo's framework: https://github.com/xxx/yyy

Audit local Python environment - show interpreters and installed libraries

Check if torch is installed

Check if local framework fits this project (current dir has implementation)

My OpenClaw MCP is broken, help me research

Research missing local capabilities for this repo
```

## Directory Structure

```
cobbler-skill/
├── SKILL.md                     # Main skill file
├── README.md                   # This file
├── README_EN.md                # English version
├── references/
│   ├── repo-analysis-checklist.md
│   ├── local-plugin-coupling.md
│   ├── mcp-cli-debug-checklist.md
│   ├── skill-maintenance-checklist.md
│   └── python-env-checklist.md
└── agents/
    └── evaluator.md            # Core evaluation logic
```

## Scope

**Cobbler Skill DOES:**
- ✅ GitHub/GitLab/Gitee repository parsing and framework analysis
- ✅ Python/Conda environment audit and library management
- ✅ Local tool suitability evaluation
- ✅ Bug diagnosis and fix plan generation
- ✅ Capability gap identification and supplement suggestions

**Cobbler Skill DOES NOT:**
- ❌ Directly modify user local files (requires user confirmation)
- ❌ Execute destructive operations (delete environment, uninstall packages)
- ❌ Access unauthorized repositories or APIs
- ❌ Generate compilable code (scripts can be generated, compilation requires user execution)

## Referenced Project

Inspired by [repo2skill](https://github.com/zhangyanxs/repo2skill) repository parsing approach. Different focus: repo2skill converts repos to Skills, Cobbler focuses on cross-repository Agent framework research.

## License

MIT License