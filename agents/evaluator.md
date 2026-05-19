# Cobbler Agent Evaluator

Core evaluation logic for the Cobbler Skill.

## Role

The Cobbler evaluator analyzes repositories, local frameworks, bugs, and capabilities to provide actionable integration plans for coding agents. Works with all major Coding Agents (OpenClaw / Codex / Claude Code / Cursor), no agent lock-in.

## Input Contract

| Framework | Input | Required Fields |
| --------- | ----- | --------------- |
| Framework 1 (Parse Repo) | GitHub repository URL | `github_url` |
| Framework 2 (Python Env Audit) | Target library name (optional, full audit if omitted) | `target_library` (optional) |
| Framework 3 (Local Suitability) | GitHub URL + local implementation path | `github_url`, `local_path` |
| Framework 4 (Bug Diagnosis) | Error logs or failure messages | `error_logs` or relevant context |
| Framework 5 (Capability Supplement) | Target capability requirements | `capability_requirement` or target repo |

## Output Contract

All frameworks output structured reports with these mandatory sections:

```text
1. 诊断 (Diagnosis)
   — What is the current state?
   — What problem exists?

2. 研究计划 (Research Plan)
   — What steps will be taken?
   — What tools will be used?

3. 具体修复方案 (Concrete Patches)
   — Exact changes to make (file + line + content)
   — Code diffs or configuration changes

4. 验证命令 (Verification Commands)
   — How to confirm the fix works?
   — Commands to run and expected output

5. 冲突和耦合说明 (Conflict and Coupling Notes)
   — What else is affected?
   — Potential conflicts with existing tools

6. 残余风险 (Remaining Risks)
   — What could go wrong?
   — What needs manual follow-up?
```

## Evaluation Criteria by Framework

### Framework 1: Repository Analysis

| Criterion | Score | Description |
| --------- | ----- | ----------- |
| Framework Type | CLI/MCP/Skill/Hybrid | Primary integration surface |
| Code Quality | High/Medium/Low | Source code organization |
| Documentation | Complete/Partial/Missing | SKILL.md, README quality |
| Compatibility | Native/Partial/Unsupported | Codex, Claude Code, OpenClaw support |
| Coupling Risk | Low/Medium/High | Potential conflicts |

**Output**: Structured analysis report with framework type, entry points, dependencies, and agent readiness assessment.

### Framework 2: Python Environment Audit & Library Management

| Criterion | Score | Description |
| --------- | ----- | ----------- |
| Interpreter Coverage | Complete/Partial/Missing | All Python envs detected |
| Library Accuracy | High/Medium/Low | Version info accuracy |
| Recommendation Quality | Strong/Weak | Best interpreter for task |
| Safety | Confirmed/Pending | User confirmation before install |

**Output**: Available interpreters with versions, installed libraries per interpreter, target library status (✅ installed / ❌ missing), recommended interpreter for installation, installation confirmation request before proceeding.

### Framework 3: Local Framework Suitability

| Criterion | Verdict |
| --------- | ------- |
| Framework Match | ✅ Suitable / ⚠️ Needs modification / ❌ Not suitable |
| Functional Overlap | None / Partial / Complete |
| Version Compatibility | Compatible / Conflict / Unknown |
| Integration Effort | Low / Medium / High |

**Output**: Suitability verdict with design intent match, functional conflicts, version compatibility, and modification suggestions.

### Framework 4: Bug Diagnosis

| Failure Layer | Description |
| ------------- | ----------- |
| missing dependency | Package not installed |
| wrong Python environment | Python version mismatch |
| wrong Node version | Node version mismatch |
| wrong working directory | CWD issue |
| wrong entrypoint | Command not found |
| missing environment variable | Env var not set |
| Windows path quoting issue | Path with spaces |
| stdio polluted by logs | Log pollution |
| process exits immediately | Server crashes |
| API key missing | Auth failure |
| permission issue | Access denied |
| file path access issue | Path not found |
| port conflict | Port in use |
| MCP config command/args mismatch | Config error |

**Output**: Root cause, patch pattern, verification command, remaining risk.

### Framework 5: Capability Supplement

**Output**: Missing capabilities, recommended tool combinations, bridging scripts, configuration updates, transformation steps.

## Quality Standards

1. **Specificity** — Each fix includes executable commands or file paths
2. **Verifiability** — Each fix has a corresponding verification command
3. **Honesty** — Unable to execute verification must state reason
4. **Completeness** — Report must include conflict and risk sections
5. **Safety** — Python environment modifications require explicit user confirmation

## Scope Boundaries

Cobbler Skill does:
- ✅ GitHub/GitLab/Gitee repository parsing and framework analysis
- ✅ Python/Conda environment audit and library management
- ✅ Local tool suitability evaluation
- ✅ Bug diagnosis and fix plan generation
- ✅ Capability gap identification and supplement suggestions

Cobbler Skill does NOT:
- ❌ Directly modify user local files (requires user confirmation)
- ❌ Execute destructive operations (delete environment, uninstall packages)
- ❌ Access unauthorized repositories or APIs
- ❌ Generate compilable code (scripts can be generated, compilation requires user execution)
- ❌ Lock to a specific Agent (works with all major Coding Agents)

## Usage Pattern

```
Use $cobbler-skill to analyze this GitHub repository and produce an integration plan.
Use $cobbler-skill to audit local Python environment.
Use $cobbler-skill to check if torch is installed.
Use $cobbler-skill to evaluate whether my local framework fits this project.
Use $cobbler-skill to debug why this MCP server exits immediately.
Use $cobbler-skill to supplement missing capabilities for this agent.
```