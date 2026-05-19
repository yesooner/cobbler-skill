---
name: cobbler-skill
description: |
  Cobbler（修鞋匠）— AI Agent 框架研究与优化 Skill
  适用所有主流 Coding Agent（OpenClaw / Codex / Claude Code / Cursor 等），不绑定特定 Agent。
  
  触发场景（当用户描述以下需求时激活）：
  - "研究这个仓库" / "分析这个 GitHub 项目" → 激活框架一
  - "检查 Python 环境" / "审计本地 Python 环境" / "这个库安装了没" → 激活框架二
  - "本地框架是否适合这个项目" / "检查本地适配度" → 激活框架三
  - "报错了" / "MCP 启动失败" / "工具出错" → 激活框架四
  - "缺少这个能力" / "大模型做不到什么" → 激活框架五
trigger:
  # 框架一触发
  - "研究这个仓库"
  - "分析这个 GitHub 项目"
  - "帮我看看这个 repo 的结构"
  - "这个仓库是什么框架"
  - "parse github"
  - "分析仓库"
  # 框架二触发
  - "检查 Python 环境"
  - "审计本地 Python 环境"
  - "这个库安装了没"
  - "帮我安装 xxx"
  - "python 环境"
  - "check env"
  - "pip list"
  - "查看有哪些 Python 解释器"
  - "检查 torch 是否已安装"
  - "帮我看看 pandas 版本"
  # 框架三触发
  - "本地框架是否适合这个项目"
  - "我的实现能跑这个仓库的 skill 吗"
  - "检查本地适配度"
  - "框架适配"
  - "check suitability"
  # 框架四触发
  - "报错了"
  - "MCP 启动失败"
  - "CLI 命令找不到"
  - "工具出错"
  - "debug"
  - "诊断问题"
  - "修复 bug"
  # 框架五触发
  - "缺少这个能力"
  - "大模型做不到什么"
  - "补充能力"
  - "能力缺失"
  - "supplement capability"
  - "缺少工具"
goal: "帮助用户研究 GitHub 仓库框架、审计 Python 环境、评估本地工具适配度、诊断 Bug、补充能力缺失"
required_inputs:
  - name: github_url
    type: string
    description: GitHub 仓库 URL（框架一/三需要）
    required: false
  - name: local_path
    type: string
    description: 本地实现路径（框架三需要）
    required: false
  - name: target_library
    type: string
    description: 目标库名称（框架二可选，不填则全面审计）
    required: false
  - name: error_logs
    type: string
    description: 错误日志或失败信息（框架四需要）
    required: false
  - name: capability_requirement
    type: string
    description: 目标能力需求描述（框架五需要）
    required: false
version: "0.4.0"
tags: [agent, evaluation, framework, mcp, skill, debugging, python, openclaw, claude-code, codex, cursor]
author: Anyeson
license: MIT
platform: any
---

# 修鞋匠 Cobbler Skill

让 AI 成为你的 Skill 研究助手。

## 五个框架

### 🔍 框架一：解析 GitHub 仓库

传入 GitHub URL，深入研究并产出结构化分析报告。

**输入**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| github_url | string | ✅ | GitHub 仓库 URL（如 `https://github.com/owner/repo`） |

**工作流：**
1. 解析 URL，提取 owner/repo 信息
2. 使用 `ask_repo_ai` 工具获取仓库结构（`get_repo_structure`）
3. 读取关键文件：README.md、package.json、pyproject.toml、SKILL.md、MCP 配置文件
4. 识别框架类型（Plan-and-Execute、MRKL、ReAct、Toolformer、Hugging Face Agent 等）
5. 检查 Skill/MCP/CLI 配置入口
6. 输出结构化分析报告

**输出格式**：
```text
仓库：owner/repo
框架类型：Agent框架类型
入口文件：主要入口
工具配置：Skill/MCP/CLI方式
依赖要求：版本和环境
初步结论：该仓库适合用来做xxx
```

**失败场景**：
- GitHub 仓库不存在或私有 → 明确告知用户仓库不可访问
- 仓库无 README 或关键配置文件 → 仍产出分析，但标记"信息不完整"
- 网络超时 → 说明无法获取完整信息，提供已获取的部分分析

**触发条件**（以下任一表述都触发）：
- "研究这个仓库"
- "分析这个 GitHub 项目"
- "帮我看看这个 repo 的结构"
- "这个仓库是什么框架"
- "parse github"
- "分析仓库"

### 🐍 框架二：Python 环境检索与库管理

全面审计本地 Python 环境，避免重复创建库或创建冲突的虚拟环境。

**输入**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| target_library | string | ❌ | 目标库名称（不填则全面审计） |

**工作流：**
1. 检测系统中所有 Python 解释器（py -0 + 常见路径 + anaconda/miniconda/pyenv）
2. 对每个解释器获取已安装的核心库版本
3. 检查目标库是否已存在（pip list / conda list）
4. 若库不存在：给出推荐解释器 + 安装命令
5. 若需安装新库：**必须与人交互确认**，确认时说明目标环境和影响范围
6. 始终优先复用已有环境，不重复创建

**输出格式**：
```text
可用解释器：
- C:\Python312\python.exe (Python 3.12.0, pip 24.0)
- C:\Users\pc\anaconda3\python.exe (Python 3.11.5, conda 23.3.1)
- C:\Users\pc\miniconda3\envs\py310\python.exe (Python 3.10.13)

核心库版本：
- numpy: ✅ 1.26.4 (C:\Python312)
- pandas: ✅ 2.0.3 (C:\Python312)
- torch: ❌ 未安装

目标库状态：
- transformers: ❌ 未安装（推荐解释器: C:\Python312）
- numpy: ✅ 1.26.4 (C:\Python312)

安装建议：
在 C:\Python312 中安装：transformers==4.40.0

交互确认：[等待用户输入 yes/no]
```

**交互确认模板**：
```text
即将在 Python 环境 C:\Python312\python.exe 中安装：
- pandas==2.0.0
此操作将修改该解释器的环境。
是否确认继续？（输入 "yes" 确认，"no" 取消）
```

**失败场景**：
- 无法找到任何 Python 解释器 → 说明系统中未检测到 Python，建议用户安装
- 权限不足无法读取某些环境 → 跳过该环境，说明检测到但无法访问
- 目标库存在但版本不兼容 → 说明已安装但版本不符合需求，提供升级方案

**触发条件**（以下任一表述都触发）：
- "检查 Python 环境"
- "审计本地 Python 环境"
- "这个库安装了没"
- "帮我安装 xxx"
- "python 环境"
- "check env"
- "pip list"
- "查看有哪些 Python 解释器"
- "检查 torch 是否已安装"
- "帮我看看 pandas 版本"

### ⚖️ 框架三：研究本地框架适配度

传入 GitHub 仓库 + 本地已有实现，研究适配程度。

**输入**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| github_url | string | ✅ | GitHub URL 或已分析的仓库结论 |
| local_path | string | ✅ | 本地实现路径（通常是当前工作目录的某个子目录） |

**工作流：**
1. 读取本地实现的 SKILL.md、AGENTS.md、配置文件
2. 对比目标仓库的框架设计与本地实现的匹配度
3. 检查功能重叠和冲突（参考 `local-plugin-coupling.md`）
4. 评估版本兼容性
5. 给出明确结论：✅ 适合 / ⚠️ 需改造 / ❌ 不适合

**输出格式**：
```text
适配度评估：[适合/需改造/不适合]
设计意图匹配：描述
功能冲突：列出冲突点（无则为"无"）
版本兼容：兼容版本范围
改造建议：如果需改造，列出具体步骤
```

**失败场景**：
- 本地路径不存在 → 明确告知用户路径无效
- 仓库信息不完整 → 基于已有信息给出初步评估，明确说明信息缺失
- 无法确定版本兼容性 → 说明无法确定，列出需要确认的项

**触发条件**（以下任一表述都触发）：
- "本地框架是否适合这个项目"
- "我的实现能跑这个仓库的 skill 吗"
- "检查本地适配度"
- "框架适配"
- "check suitability"

### 🔧 框架四：研究大模型适配 Bug

诊断任意 Agent 调用本地 Skill/MCP/CLI 时的问题。

**输入**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| error_logs | string | ✅ | 错误信息、失败日志、MCP 配置内容 |

**工作流：**
1. 识别失败层级（依赖缺失 / 环境错误 / 路径问题 / 配置不匹配）
2. 提取最小复现命令
3. 对照 `mcp-cli-debug-checklist.md` 分类
4. 定位根因
5. 给出最小修复方案

**失败分类**：
```text
missing dependency              — 缺少依赖包
wrong Python environment       — Python 环境错误
wrong Node version             — Node 版本错误
wrong working directory        — 工作目录错误
wrong entrypoint               — 入口文件错误
missing environment variable   — 环境变量缺失
Windows path quoting issue     — Windows 路径引号问题
stdio polluted by logs         — 日志污染标准输出
process exits immediately      — 进程立即退出
API key missing                — API 密钥缺失
permission issue               — 权限问题
file path access issue         — 文件路径访问问题
port conflict                  — 端口冲突
MCP config command/args mismatch — MCP 配置命令/参数不匹配
```

**输出格式**：
```text
根因：[一句话描述]
失败层级：[分类]
修复方案：[具体修改]
验证命令：[可执行的验证命令]
残余风险：[可能的遗留问题]
```

**失败场景**：
- 错误日志不完整 → 明确说明缺少哪些信息，列出需要补充的内容
- 无法复现问题 → 说明无法复现，基于日志给出可能的原因
- 修复后验证失败 → 说明修复未成功，提供残余风险

**触发条件**（以下任一表述都触发）：
- "报错了"
- "MCP 启动失败"
- "CLI 命令找不到"
- "工具出错"
- "debug"
- "诊断问题"
- "修复 bug"

### 🩹 框架五：研究能力补充

研究本地工具生态中大模型原生不具备的能力。

**输入**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| capability_requirement | string | ✅ | 目标仓库或任务描述 |

**工作流：**
1. 分析目标任务需要但本地工具缺失的能力
2. 推荐可补充的 Skill 配置
3. 生成桥接脚本或适配层代码
4. 提出 MCP server 补充配置
5. 输出改造步骤代码块

**输出格式**：
```text
缺失能力：[描述]
补充方案：[推荐的工具组合]
桥接代码：[脚本或适配器]
配置更新：[MCP 或 Skill 配置]
改造步骤：[分步骤说明]
```

**失败场景**：
- 目标能力已存在但用户不知道 → 说明现有工具已满足需求
- 无法找到合适的补充方案 → 说明当前工具生态无法满足，列出原因
- 补充方案需要外部依赖 → 说明依赖及其获取方式

**触发条件**（以下任一表述都触发）：
- "缺少这个能力"
- "大模型做不到什么"
- "补充能力"
- "能力缺失"
- "supplement capability"
- "缺少工具"

## 输出约定

使用以下结构（除非用户要求其他格式）：

```text
1. 诊断
2. 研究计划
3. 具体修复方案或生成的配置文件
4. 验证命令
5. 冲突和耦合说明
6. 残余风险
```

### 输出质量要求

- **具体性**：每个修复方案必须包含可执行的命令或文件路径
- **可验证**：每个修复方案必须有对应的验证命令
- **诚实性**：无法执行的验证必须说明原因
- **完整性**：报告必须包含冲突说明和残余风险

### 失败处理

当任务超出 Skill 范围时：
- 明确告知用户超出范围的具体原因
- 建议可能的替代路径
- 不返回模糊或猜测性的结论

### 工具边界

本 Skill 专注于：
- ✅ GitHub/GitLab/Gitee 仓库解析和框架分析
- ✅ Python/Conda 环境全面审计和库管理
- ✅ 本地工具适配度评估
- ✅ Bug 诊断和修复方案生成
- ✅ 能力缺失识别和补充建议

本 Skill 不做：
- ❌ 直接修改用户本地文件（需用户确认）
- ❌ 执行破坏性操作（如删除环境、卸载包）
- ❌ 访问未授权的仓库或 API
- ❌ 生成需要编译的代码（生成脚本可以，但编译需用户执行）
- ❌ 绑定特定 Agent（适用于所有主流 Coding Agent）

## 参考文件

按需加载以下文件：
- `references/repo-analysis-checklist.md` — GitHub 仓库分析方法
- `references/local-plugin-coupling.md` — 本地工具耦合分析
- `references/mcp-cli-debug-checklist.md` — Bug 诊断方法
- `references/skill-maintenance-checklist.md` — Skill 维护规则
- `references/python-env-checklist.md` — Python 环境检索方法

## 测试提示

使用以下提示验证 Skill 各框架：

### 框架一测试
```text
帮我研究这个仓库的 Agent 框架：https://github.com/openclaw/openclaw
帮我分析这个 GitHub 项目：https://github.com/anysphere/cursor
这个仓库是什么框架？帮我看看 https://github.com/zhangyanxs/repo2skill
```

### 框架二测试
```text
帮我审计本地 Python 环境，看看有哪些解释器和已装库
检查 torch 是否已安装
帮我看看 pandas 版本
这个库安装了没？transformers
查看有哪些 Python 解释器
pip list 看看已安装的包
审计 Python 环境：检查 numpy, torch, transformers 状态
```

### 框架三测试
```text
帮我研究本地框架是否适合这个项目（当前目录已有实现）
我的 OpenClaw 实现能否跑这个仓库的 skill？
检查本地适配度：当前目录 vs https://github.com/xxx/yyy
```

### 框架四测试
```text
我的 OpenClaw 调用这个 MCP 报错了：
[错误日志]
帮忙研究一下是什么问题

MCP 启动失败，进程立即退出，帮我诊断

CLI 命令找不到，报错 'command not found'，帮忙看看
```

### 框架五测试
```text
帮我研究这个仓库缺失的本地能力
这个项目需要什么能力？本地工具生态有没有缺失？
补充能力：https://github.com/xxx/yyy 缺少什么工具？
```

### 综合测试
```text
使用所有框架进行完整测试：
1. 研究一个 GitHub 仓库
2. 审计 Python 环境
3. 评估本地框架适配度
4. 诊断一个模拟 bug
5. 识别缺失能力
```

## 目录结构

```
cobbler-skill/
├── SKILL.md                     # 主技能文件
├── README.md                   # 中文说明
├── README_EN.md                # English version
├── references/
│   ├── repo-analysis-checklist.md
│   ├── local-plugin-coupling.md
│   ├── mcp-cli-debug-checklist.md
│   ├── skill-maintenance-checklist.md
│   └── python-env-checklist.md
└── agents/
    └── evaluator.md            # 核心评估逻辑
```

## 参考项目

本 Skill 参考 [repo2skill](https://github.com/zhangyanxs/repo2skill) 仓库解析思路构建，但定位不同：repo2skill 侧重将仓库转换为 Skill，本 Skill 侧重跨仓库的 Agent 框架研究与互修。

## 许可

MIT License