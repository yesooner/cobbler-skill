# Local Plugin Coupling Analysis

Use this reference when several local tools, plugins, skills, or MCP servers may overlap or conflict.

## When to Use

Use this checklist in these scenarios:
- User has multiple tools that can implement the same function
- New project needs integration into existing toolchain
- Discovering tool conflicts or duplication
- Planning tool division of labor and boundaries

## Coupling Matrix Construction

Build the following table to evaluate tool relationships:

| Tool | Role | Input | Output | Interface | State | Risk | Coupling |
| ---- | ---- | ----- | ------ | --------- | ----- | ---- | --------- |
|      |      |       |        |           |       |      |          |

### Field Description

| Field | Description | Example |
| ----- | ----------- | ------- |
| Tool | Tool name | `paper-fetch-skill` |
| Role | Role: retrieval/conversion/citation/editing | `retrieval` |
| Input | Input format and data source | `DOI string` |
| Output | Output format and destination | `PDF file path` |
| Interface | Interface type: CLI/MCP/Skill/Native API | `MCP` |
| State | State: stateless/stateful | `stateless` |
| Risk | Risk level: High/Medium/Low | `Low` |
| Coupling | Coupling: loose/tight/none | `loose` |

## Local Tool Classification Reference

When analyzing, consider these common tool types:

### Retrieval Tools (检索工具)

```text
repo2skill         — GitHub repository to Skill converter
OneFind           — Local file search
paper-fetch-skill — Paper fetching
MinerU            — PDF content extraction
Zotero MCP        — Literature retrieval via Zotero
```

### Conversion Tools (转换工具)

```text
Docling          — Document structuring (PDF/Word → structured)
Marker           — PDF to Markdown (general purpose)
GROBID           — Academic PDF parsing (citations, references)
Mermaid CLI      — Diagram rendering (mermaid → PNG/SVG)
Pandoc           — Universal document converter
```

### Citation Tools (引用工具)

```text
Zotero Better BibTeX — Literature management + BibTeX export
Obsidian citation     — Note citation plugin
Better BibTeX         — Citation key generation
```

### Editing Tools (编辑工具)

```text
Claude Code     — Code editing (general purpose)
Codex           — Code generation (general purpose)
openclaw        — General Agent (multi-purpose)
Cursor          — AI-powered code editor
```

## Conflict Type Identification

Check for these conflict patterns:

```text
输出目录冲突    — Two tools write to the same output directory
                  → Solution: Assign different output directories

端口冲突        — Two MCP servers use the same port (e.g., 3000)
                  → Solution: Configure different ports

命令名冲突      — Different packages provide the same CLI command name
                  → Solution: Use full path or uninstall duplicate

环境冲突        — Same workflow needs different Python environments
                  → Solution: Use separate virtual environments

日志污染        — stdout logs break stdio MCP protocol
                  → Solution: Redirect logs to stderr

缓存冲突        — Shared cache schema incompatible between tools
                  → Solution: Use isolated cache directories

源文件修改      — Tool accidentally modifies source files
                  → Solution: Work on copies, verify with diff

授权冲突        — Licensed tool used in unsupported scenario
                  → Solution: Check license terms, use alternatives

网络依赖        — Offline workflow depends on network
                  → Solution: Document network requirements

技能重复        — Multiple skills have overlapping trigger descriptions
                  → Solution: Disambiguate triggers, establish priority
```

## Coupling Analysis Workflow

### Step 1: Identify All Tools

List all tools in the local ecosystem that could potentially overlap.

### Step 2: Classify by Role

Assign each tool to a role category (retrieval/conversion/citation/editing).

### Step 3: Build Coupling Matrix

Create the coupling matrix showing relationships between tools.

### Step 4: Identify Conflicts

Detect any conflicts based on the conflict patterns listed above.

### Step 5: Recommend Integration

Based on the analysis, recommend how tools should be integrated or separated.

## Coupling Guidelines

### Prefer Loose Coupling

For batch workflows, prefer file-based or CLI output for tool integration:
- Tool A outputs JSON file
- Tool B reads that file as input
- Don't share memory or state directly

### Use MCP Only When Needed

Use MCP only when Agent needs repeated structured calls:
- Single command execution → CLI is sufficient
- Multiple queries/calls → MCP is appropriate
- Complex state management → Need dedicated Agent

### Require Confirmation for Destructive Operations

For operations that may modify data:
- Always require explicit user confirmation
- Record original data location for recovery
- Distinguish "suggestion" vs "required"

## Tool Boundary Definition

Each tool should clearly define:

1. **Input Boundary**: What format and source it accepts
2. **Output Boundary**: What format it produces and where it goes
3. **Side Effects**: Whether it modifies original data
4. **Dependencies**: What prerequisite tools or environment it needs
5. **Conflicts**: Which tools it cannot coexist with

## Coupling Analysis Output Template

```text
工具链：[List all related tools]
工具数量：[N] 个

角色分工：
| 工具 | 角色 | 输入 | 输出 | 接口 | 状态 | 风险 | 耦合 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      |      |      |      |      |      |      |

冲突检测：
| 冲突类型 | 涉及工具 | 描述 | 解决方案 |
| -------- | -------- | ---- | -------- |
|        |          |      |          |

推荐集成方案：
1. [Step 1]
2. [Step 2]
...

风险评估：
| 风险 | 可能性 | 影响 | 缓解措施 |
| ---- | ------ | ---- | -------- |
|      |        |      |          |
```

## Decision Rules

1. **Use CLI when command is sufficient** — Don't introduce MCP for simple command calls
2. **Don't mark as agent-ready unless startup command is known** — Verification commands must be known
3. **Prefer source files** — Don't use generated artifacts as primary navigation targets
4. **Keep public analysis separate from private assumptions** — GitHub analysis doesn't include private repo assumptions
5. **Prioritize loose coupling** — Tight coupling leads to fragile systems
6. **Document all conflicts** — Undocumented conflicts cause production issues