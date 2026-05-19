# 修鞋匠 Cobbler Skill

> 让 AI 成为你的 Skill 研究助手
> 适用所有主流 Coding Agent（OpenClaw / Codex / Claude Code / Cursor 等），不绑定特定 Agent。

<p align="center">
  <strong>📝 其他语言 / Other Languages:</strong><br>
  中文 |
  <a href="./i18n/README.en.md">English</a>
</p>

## 是什么

Cobbler 是一个帮助研究和优化本地 Skill/MCP/CLI 工具的 Skill，帮助你：

- 解析 GitHub/GitLab/Gitee 仓库，理解其工具生态和框架设计
- **本地 Python 环境审计与库管理**，避免重复创建环境
- 研究本地工具与目标项目的适配程度
- 诊断并修复任意 Agent 调用本地工具的适配问题
- 研究并补充大模型缺失的本地能力

## 五个框架

### 🔍 框架一：解析 GitHub 仓库

传入 GitHub URL，深入研究并产出结构化分析报告：

- 项目结构、入口文件、核心模块
- Agent 框架类型（Plan-and-Execute、MRKL、ReAct、Toolformer 等）
- Skill / MCP / CLI 配置方式
- 依赖和版本要求

**触发条件**（以下任一表述都触发）：
- "研究这个仓库"
- "分析这个 GitHub 项目"
- "帮我看看这个 repo 的结构"
- "这个仓库是什么框架"
- "parse github"
- "分析仓库"

### 🐍 框架二：Python 环境检索与库管理

全面审计本地 Python 环境，避免重复创建库或创建冲突的虚拟环境。

- 列出所有可用 Python 解释器路径和版本
- 列出每个解释器中已安装的核心库版本
- 检查目标库是否已安装
- 若需安装新库，**必须与人交互确认**，确保环境干净
- 始终优先复用已有环境，不重复创建

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

传入 GitHub 仓库 + 本地已有实现，研究适配程度：

- 本地框架是否匹配目标仓库的设计意图？
- 是否有功能冲突或能力缺失？
- 版本是否兼容？
- 给出明确结论：✅ 适合 / ⚠️ 需改造 / ❌ 不适合

**触发条件**（以下任一表述都触发）：
- "本地框架是否适合这个项目"
- "我的实现能跑这个仓库的 skill 吗"
- "检查本地适配度"
- "框架适配"
- "check suitability"

### 🔧 框架四：研究大模型适配 Bug

诊断 Codex / Claude Code / OpenClaw 调用本地工具时的问题：

- Skill：SKILL.md 格式错误、触发条件不明确、路径问题
- MCP：工具注册失败、协议不匹配、环境变量错误
- CLI：命令找不到、参数错误、Windows 路径问题
- 定位根因，给出诊断和修复方案

**触发条件**（以下任一表述都触发）：
- "报错了"
- "MCP 启动失败"
- "CLI 命令找不到"
- "工具出错"
- "debug"
- "诊断问题"
- "修复 bug"

### 🩹 框架五：研究能力补充

研究本地工具生态中大模型原生不具备的能力：

- 推荐需要补充的 Skill 配置
- 生成桥接脚本或适配层代码
- 提出 MCP server 补充配置
- 生成改造步骤代码块

**触发条件**（以下任一表述都触发）：
- "缺少这个能力"
- "大模型做不到什么"
- "补充能力"
- "能力缺失"
- "supplement capability"
- "缺少工具"

## 输出约定

所有框架输出包含以下强制部分：

```text
1. 诊断 — 当前状态是什么？存在什么问题？
2. 研究计划 — 将采取什么步骤？使用什么工具？
3. 具体修复方案 — 精确的修改内容（文件 + 行号 + 内容）
4. 验证命令 — 如何确认修复有效？
5. 冲突和耦合说明 — 还影响什么？
6. 残余风险 — 可能出现什么问题？
```

## 输出质量标准

- **具体性**：每个修复方案必须包含可执行的命令或精确的文件路径
- **可验证**：每个修复方案必须有对应的验证命令
- **诚实性**：无法执行的验证必须说明原因
- **完整性**：报告必须包含冲突说明和残余风险

## 快速开始

```text
帮我研究这个仓库的框架：https://github.com/xxx/yyy

帮我审计本地 Python 环境，看看有哪些解释器和已装库

检查 torch 是否已安装

帮我研究本地框架是否适合这个项目（当前目录已有实现）

我的 OpenClaw 调用这个 MCP 报错了，帮我研究一下

帮我研究这个仓库缺失的本地能力
```

## 目录结构

```
cobbler-skill/
├── SKILL.md                     # 主技能文件
├── README.md                   # 本文件
├── i18n/
│   └── README.en.md            # English version
├── references/
│   ├── repo-analysis-checklist.md
│   ├── local-plugin-coupling.md
│   ├── mcp-cli-debug-checklist.md
│   ├── skill-maintenance-checklist.md
│   └── python-env-checklist.md
└── agents/
    └── evaluator.md            # 核心评估逻辑
```

## Skill 范围

**本 Skill 做：**
- ✅ GitHub/GitLab/Gitee 仓库解析和框架分析
- ✅ Python/Conda 环境全面审计和库管理
- ✅ 本地工具适配度评估
- ✅ Bug 诊断和修复方案生成
- ✅ 能力缺失识别和补充建议

**本 Skill 不做：**
- ❌ 直接修改用户本地文件（需用户确认）
- ❌ 执行破坏性操作（如删除环境、卸载包）
- ❌ 访问未授权的仓库或 API
- ❌ 生成需要编译的代码（生成脚本可以，编译需用户执行）

## 参考项目

本 Skill 参考 [repo2skill](https://github.com/zhangyanxs/repo2skill) 仓库解析思路构建，但定位不同：repo2skill 侧重将仓库转换为 Skill，本 Skill 侧重跨仓库的 Agent 框架研究与互修。

## 许可

MIT License
