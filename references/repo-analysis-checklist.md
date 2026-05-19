# Repository Analysis Checklist

Use this reference when analyzing a local or GitHub repository for agent use.

## When to Use

Use this checklist when:
- User asks to "research this repo" or "analyze this GitHub project"
- Evaluating if a repository is suitable for agent integration
- Determining what framework type a project uses
- Assessing tool configuration patterns (Skill/MCP/CLI/Native)

## Repository Type Classification

Classify the project into one or more types:

```text
CLI tool                        — Command-line tool
MCP server                      — MCP server implementation
Codex skill                     — Codex Skill format
Claude Code subagent            — Claude Code subagent
Claude slash command            — Claude Code slash command
Node package                    — Node.js package
Python package                  — Python package
desktop app                     — Desktop application
API service                     — API/web service
PDF/parser tool                 — PDF parsing tool
Zotero/Obsidian connector       — Literature connector
paper-fetch/downloader          — Paper fetching tool
workflow-only prompt repository — Prompt-only workflow
Agent framework                 — Agent architecture (Plan-and-Execute, MRKL, ReAct, Toolformer, etc.)
```

## Evidence Extraction

Record the following information:

```text
package manager                 — Package manager (npm, pip, conda, pyproject.toml)
runtime and version constraints  — Runtime and version constraints
CLI commands                    — CLI command entry points
source entrypoints              — Source code entry points
published package entrypoints    — Published package entry points
MCP server commands              — MCP server startup commands
configuration files             — Configuration files
environment variables            — Required environment variables
important directories           — Important directories (source, config, data)
generated artifacts              — Generated artifacts (dist/, build/, .venv/)
validation commands              — Validation commands (test, build, health check)
known unsupported cases         — Known unsupported scenarios
agent framework type            — Agent framework type
```

## Analysis Workflow

### Step 1: Structure Scan

Use `ask_repo_ai` tool's `get_repo_structure` to get the repository tree structure.

### Step 2: Key File Reading

Read files in priority order:
1. README.md — Project overview and quick start
2. package.json / pyproject.toml / setup.py — Dependencies and entry points
3. SKILL.md — If present, indicates Agent-designed project
4. AGENTS.md — If present, Agent behavior specification
5. .github/workflows/*.yml — CI/CD configuration (reveals build and test process)
6. src/ or lib/ directory structure — Core code structure
7. mcp*.json or similar MCP configuration files

### Step 3: Framework Type Identification

Determine framework type based on file contents:

| Feature | Framework Type |
| ------- | --------------- |
| Heavy tool definitions | Toolformer |
| plan + execute separation | Plan-and-Execute |
| Mixed tool + reasoning | MRKL / ReAct |
| huggingface agent imports | Hugging Face Agent |
| state machine + transition rules | Finite State Machine Agent |

### Step 4: Tool Configuration Detection

Detect the tool type used by the project:

```text
Skill:  SKILL.md or .skill/ directory exists
MCP:    mcp.json or mcp-server related config exists
CLI:    bin/ or package.json bin field, or pyproject.toml console_scripts
Native: Direct source code invocation, no intermediate layer
```

## Framework Type Detection Decision Tree

Check in the following order to determine project framework type:

```
START
├── Project has SKILL.md or .skill/ directory?
│   └── YES → Skill-based project (agent-designed)
│
├── Project has heavy tool definitions or tool use code?
│   ├── YES → Possibly Toolformer or ReAct framework
│   │   └── Check for plan/act separation → Plan-and-Execute
│   └── NO → Continue
│
├── Project has Agent class inheritance or hf.agent imports?
│   ├── YES → Hugging Face Agent
│   └── NO → Continue
│
├── Project has state machine definitions or transition rules?
│   ├── YES → Finite State Machine Agent
│   └── NO → Continue
│
├── Project is pure CLI without Agent logic?
│   ├── YES → CLI tool (not an Agent framework)
│   └── NO → Continue
│
└── Project has mixed tool + reasoning calls?
    ├── YES → MRKL or ReAct
    └── NO → Possibly other hybrid frameworks or Unknown
```

## Tool Configuration Detection Priority

Check configuration in this order:

```
1. Skill detection
   └── Look for: SKILL.md, .skill/, skills/ directory
   
2. MCP detection
   └── Look for: mcp.json, mcp*.json, .mcp/, mcp-server config
   
3. CLI detection
   └── Look for: bin/, package.json bin field, pyproject.toml console_scripts, setup.py entry_points
   
4. Native detection
   └── Direct source code import without intermediate layer
```

## repo2skill Baseline

`repo2skill` currently converts repository evidence into:

```text
repo2skill.json          — Structured metadata
project-map.md          — Project structure map
AGENTS.md               — Agent behavior specification
SKILL.md                — Skill definition
quickstart.windows.md   — Windows quick start
quickstart.macos.md     — macOS quick start
quickstart.linux.md     — Linux quick start
report.html             — HTML report
```

Maintain evidence-driven model: detectors collect facts, exporters render artifacts, all outputs should derive from a structured analysis object.

## Agent-Integration Extension

When optimizing repo2skill-style output for MCP/CLI/Skill work, add or recommend these artifacts:

```text
agent-integration.md     — Agent integration guide
mcp-cli-diagnosis.md    — MCP/CLI diagnosis results
skill-audit.md          — Skill audit report
coupling-matrix.md      — Coupling matrix
```

## Decision Rules

1. **Use CLI when stable CLI is sufficient** — Don't recommend MCP unnecessarily
2. **Don't mark project as "agent ready" unless startup and validation commands are known**
3. **Don't use generated `dist/` files as primary navigation targets unless source is unavailable**
4. **Keep public GitHub analysis separate from private repository assumptions**
5. **Always prefer source files over generated artifacts**

## Structured Output Template

```
仓库：[owner/repo]
类型：[Classification]
框架：[Framework type]
入口：[Entry file]
工具配置：[Skill/MCP/CLI/Native]
依赖：[Key dependencies and versions]
环境要求：[Required environment variables]
验证命令：[Command to verify project works]
不适用的场景：[Known unsupported scenarios]
Agent 就绪度：[Ready / Partial / Not Ready]

决策过程：
1. [Check result 1]
2. [Check result 2]
...

冲突和耦合说明：
[列出可能的工具冲突]

残余风险：
[列出可能的问题]
```

## Evidence Standards

When recording information, must include:

| Category | Required Fields |
| -------- | ---------------- |
| Package Manager | Package manager name + version constraint |
| Runtime | Runtime name + version range |
| CLI Entry | Command name + entry file path |
| MCP Config | Config file path + command + args |
| Environment Variables | Variable name + required + default value |

## Common Issues and Solutions

| Issue | Detection | Solution |
| ----- | --------- | -------- |
| No README | File not found | Mark as "Documentation: Missing" |
| No SKILL.md | File not found | Note as "not agent-designed" |
| Missing validation commands | No test/health check found | Mark as "Agent Readiness: Partial" |
| Only generated files available | dist/, build/ present but no src/ | Use generated artifacts, note limitation |
| Version constraints unclear | No version info in package.json | Mark as "Unknown" and note |