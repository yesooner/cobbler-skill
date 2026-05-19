# Skill Maintenance Checklist

Use this reference when creating, renaming, or rewriting a Codex (or any Agent) skill.

## When to Use

Use this checklist when:
- Creating a new skill from scratch
- Rewriting an existing skill to improve quality
- Auditing a skill for completeness and correctness
- Converting a repository to skill format

## Required Frontmatter Fields

A reliable skill has these frontmatter fields:

```yaml
---
name: skill-name                    # Unique identifier, lowercase-with-hyphens
description: |                      # Multi-line, detailed description
  # 触发面 (Trigger Surface): when to use
  # Include:
  #   - Specific use scenarios (when should user invoke this?)
  #   - What inputs it accepts
  #   - How it differs from other tools
trigger:                            # Array of trigger phrases
  - "phrase 1"
  - "phrase 2"
goal: "One sentence describing what this skill does"
required_inputs:                    # Named inputs with types
  - name: field_name
    type: string/number/boolean/object
    description: Field description
    required: true/false
workflow:                          # Ordered, executable steps
  - step: "Step description"
    command: "command to execute (if applicable)"
tool_policy:
  allowed:
    - read / write / exec
    - web_search / web_fetch
    - ask_repo_ai
  denied:
    - delete (unless user explicitly confirms)
    - external_email / external_message
    - sudo / elevated permissions
  boundary:
    - "Do not modify $HOME config files"
    - "Do not install global npm packages"
    - "Only operate in current working directory"
output_contract:                    # Structure of promised output
failure_handling:                  # Common failures and responses
  - symptom: "What goes wrong"
    response: "How to handle it"
validation:                        # How to verify skill works
  - description: "What to verify"
    command: "Verification command"
    expected: "Expected output or condition"
references:
  - references/file-name.md
---
```

## Field Description

### name

- Unique identifier for the skill
- Use lowercase with hyphens: `paper-fetch-skill`, `repo-analysis`
- Avoid special characters, spaces, uppercase

### description (Trigger Surface)

The description is the trigger surface. It should:
- Include "when to use" details, not just background
- List specific usage scenarios
- Explain how it differs from similar tools

**Good Description Example**:
```yaml
description: |
  当用户要求下载学术论文、查找 DOI、或管理 Zotero 文献时触发。
  支持 IEEE, Elsevier, Springer, Wiley, MDPI 等出版商。
  不适用于中文期刊（请使用 cnki-* 系列技能）。
```

**Bad Description Example**:
```yaml
description: |
  这是一个论文下载技能。
```

### trigger

Array of phrases that activate this skill. Include:
- Direct invocations: "download paper", "帮我下载论文"
- Implicit requests: "我想要这篇论文", "find DOI for..."
- Variations in multiple languages if applicable

### goal

One sentence that clearly describes what this skill accomplishes. Should:
- Be specific, not generic
- Exclude implementation details
- Be understandable to end users

### required_inputs

Named inputs with:
- `name`: identifier
- `type`: string/number/boolean/object
- `description`: what the field means
- `required`: true/false

### workflow

Ordered, executable steps. Each step should:
- Be concrete and actionable
- Include commands where applicable
- Be in logical order

### tool_policy

Define what this skill can and cannot do:

```yaml
tool_policy:
  allowed:
    - read / write / exec
    - web_search / web_fetch
    - ask_repo_ai
  denied:
    - delete (unless user explicitly confirms)
    - external_email / external_message
    - sudo / elevated permissions
  boundary:
    - "Do not modify $HOME config files"
    - "Do not install global npm packages"
    - "Only operate in current working directory"
```

### output_contract

Define the structure and sections that will always be present in output. Example:

```yaml
output_contract: |
  输出包含以下强制部分：
  1. 诊断 — 当前状态是什么？
  2. 研究计划 — 将采取什么步骤？
  3. 具体方案 — 精确的修改内容
  4. 验证命令 — 如何确认有效？
  5. 冲突说明 — 还影响什么？
  6. 残余风险 — 可能出现什么问题？
```

### failure_handling

Common failures and how to handle them:

```yaml
failure_handling:
  - symptom: "GitHub 仓库不存在"
    response: "明确告知用户仓库不可访问，提供可能的原因"
  - symptom: "无法获取关键文件"
    response: "仍产出分析，但标记信息不完整"
```

### validation

How to verify the skill works correctly:

```yaml
validation:
  - description: 验证 Python 环境
    command: python --version && pip list | grep -E "requests|beautifulsoup"
    expected: "Python 3.10" in output
  
  - description: 验证 MCP 服务器启动
    command: node dist/mcp-server.js --help
    expected: 显示帮助信息
```

## Common Problems and Fixes

| Problem | Fix |
| ------- | --- |
| description too vague | Add specific use scenarios and trigger phrases |
| missing trigger conditions | Add trigger array in frontmatter |
| workflow not executable | Change to specific steps with command examples |
| missing input requirements | Clarify required_inputs |
| missing output contract | Add output_contract |
| missing validation steps | Add validation and verification commands |
| too much theory | Remove explanatory text, keep operational steps |
| unsafe assumptions | Clarify boundary conditions and prerequisites |
| overlaps with existing skill | Add compatibility and conflict explanation |
| unclear relationship with CLI/MCP | Add tool_policy explaining tool boundaries |
| missing compatibility notes | Add Codex/Claude Code/OpenClaw compatibility notes |

## Rewrite Rules

1. **Keep useful domain logic** — Don't delete business knowledge
2. **Remove vague language** — Change "maybe" to specific description
3. **Convert broad goals to specific steps** — Each step executable
4. **Command-level verification** — Add verification commands for code-related steps
5. **Add failure handling** — What can go wrong at each step
6. **Add tool boundaries** — Explain which tools cannot be used
7. **Add compatibility and conflict section** — Explain relationship with other tools
8. **Split long content** into independent one-hop reference files
9. **Keep SKILL.md concise** — Enough for fast loading

## Skill Quality Checklist

- [ ] frontmatter name is clear and unique
- [ ] frontmatter description includes trigger scenarios
- [ ] trigger array lists common trigger phrases
- [ ] goal is one sentence clarifying the objective
- [ ] required_inputs lists required inputs with types
- [ ] workflow each step is executable and in order
- [ ] tool_policy defines tool boundaries
- [ ] output_contract defines output format
- [ ] failure_handling lists common failures
- [ ] validation includes executable verification commands
- [ ] references relates to reference files
- [ ] no vague language (avoid "maybe", "perhaps", "probably")
- [ ] no conflict with other skills or boundaries explained
- [ ] failure scenarios are documented for each framework

## Validation Commands

Every skill involving command execution should include validation commands:

```yaml
validation:
  - description: Verify Python environment
    command: python --version && pip list | grep -E "requests|beautifulsoup"
    expected: "Python 3.10" in output
  
  - description: Verify MCP server starts
    command: node dist/mcp-server.js --help
    expected: Shows help information
```

## Skill Rewrite Workflow

### Step 1: Read Current Skill

Read the existing SKILL.md to understand current state.

### Step 2: Identify Missing Fields

Check against the required frontmatter fields.

### Step 3: Enhance Description

Add trigger surface details to description.

### Step 4: Add Trigger Array

List common trigger phrases.

### Step 5: Clarify Workflow

Make each step executable with commands.

### Step 6: Add Tool Policy

Define what can and cannot be done.

### Step 7: Define Output Contract

Specify the output structure.

### Step 8: Document Failures

Add failure handling section.

### Step 9: Add Validation

Include verification commands.

### Step 10: Review and Test

Verify the skill works correctly.