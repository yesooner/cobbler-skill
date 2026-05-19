# Python Environment Checklist

Use this reference when checking Python environments for agent use.

## When to Use

Use this checklist when:
- User asks to "check Python environment" or "audit local Python"
- User wants to know if a specific library is installed
- User wants to install a library and needs to find the right environment
- User wants to avoid recreating existing Python environments

## Goals

- Avoid recreating existing Python environments
- Keep environments clean (must obtain user confirmation before installing new libraries)
- Provide accurate Python interpreter paths and library version information

## Detection Workflow

### Step 1: Find Python Interpreters

Detect all available Python interpreters on Windows:

```powershell
# Method 1: where command
where python
where python3

# Method 2: Python Launcher (recommended)
py -0  # List all installed versions

# Method 3: Check common paths
# User install paths
C:\Python39\python.exe
C:\Python310\python.exe
C:\Python311\python.exe
C:\Python312\python.exe
C:\Users\<user>\AppData\Local\Programs\Python\Python*\python.exe

# Virtual environments
C:\Users\<user>\anaconda3\python.exe
C:\Users\<user>\anaconda3\envs\*\python.exe
C:\Users\<user>\miniconda3\python.exe
C:\Users\<user>\miniconda3\envs\*\python.exe
C:\Users\<user>\.pyenv\pyenv\versions\*\python.exe
C:\Users\<user>\AppData\Local\Programs\Python\Python*\*\python.exe

# System-level
C:\Program Files\Python*\python.exe
```

### Step 2: Get Installed Libraries

For each interpreter, execute:

```powershell
# Python version and pip version
python --version
pip --version

# Installed packages list
python -m pip list

# Check specific package version
python -c "import <package>; print(<package>.__version__)"
```

### Step 3: Check Target Library

```powershell
# Method 1: Direct check
python -c "import <library>; print('<library> installed')" 2>&1

# Method 2: pip show
pip show <library-name>

# Method 3: conda (if using conda)
conda list <library-name>
```

## Output Format

Output results in this format:

```text
可用解释器：
- C:\Python312\python.exe (Python 3.12.0, 64-bit)
- C:\Python311\python.exe (Python 3.11.5, 64-bit)
- C:\Users\pc\anaconda3\python.exe (Python 3.11.5, Anaconda)
- C:\Users\pc\miniconda3\envs\py310\python.exe (Python 3.10.13, Conda)

核心库版本：
- pip: 24.0
- setuptools: 70.0.0
- wheel: 0.43.0
- numpy: 1.26.4
- pandas: 2.0.3
- requests: 2.31.0

目标库状态：
- numpy: ✅ 1.26.4 (C:\Python312)
- pandas: ✅ 2.0.3 (C:\Python312)
- requests: ✅ 2.31.0 (C:\Python312)
- torch: ❌ 未安装
- transformers: ❌ 未安装

推荐解释器：C:\Python312\python.exe（版本最新，库最全）
```

## Pre-Installation Confirmation

When a new library needs to be installed, must use this template for interactive confirmation:

```text
即将在 Python 环境 [解释器路径] 中安装以下库：
- [库名]==[版本]
此操作将修改该解释器的环境。
是否确认继续？（输入 "yes" 确认，"no" 取消）
```

If user agrees, execute installation and verify:

```powershell
pip install <package>==<version>
python -c "import <package>; print(<package>.__version__)"
```

## Virtual Environment Management

If a new environment needs to be created (to keep environment clean):

```powershell
# Create virtual environment
python -m venv C:\path\to\new-env

# Activate environment (Windows)
.\env\Scripts\activate

# Install in virtual environment
pip install <package>
```

## Environment Detection Output Template

```text
检测完成时间：[Timestamp]
可用解释器数量：[N] 个

可用环境：
| 路径 | 版本 | 类型 | 核心库数量 |
| ---- | ---- | ---- | ---------- |
|      |      |      |            |

目标库检查：
| 库名 | 状态 | 版本 | 路径 |
| ---- | ---- | ---- | ---- |
|      |      |      |      |

安装建议：[是否需要安装 / 推荐哪个解释器]

冲突和耦合说明：
[多版本共存可能引发的问题]

残余风险：
[安装后可能的依赖冲突]
```

## Failure Scenario Handling

| Scenario | Detection | Response |
| --------- | --------- | -------- |
| No Python found | `py -0` returns empty, `where python` fails | 说明系统中未检测到 Python，建议用户安装 |
| Permission denied | Cannot read certain environment | 跳过该环境，在输出中说明"检测到但无法访问" |
| Library version incompatible | Library installed but version doesn't meet requirement | 说明已安装但版本不符合需求，提供升级方案 |
| Multiple versions exist | Same library in different interpreters | 列出所有版本及所在环境，建议使用最新稳定版 |
| Conda and pip conflict | Same library different versions in conda vs pip | 说明环境冲突，建议使用统一的包管理器 |
| Network unavailable | pip install fails due to network | 说明网络不可用，建议手动下载或检查网络设置 |

## Decision Rules

1. **Always require user confirmation** — Must confirm before installation
2. **Prioritize existing environments** — Don't recreate environments for already available libraries
3. **Record exact paths** — Use full paths, not just `python`
4. **Check multiple interpreters** — Same library may have different versions across environments
5. **Distinguish conda vs pip** — Check two package manager environments separately
6. **Prefer pip for most cases** — Conda has more complex dependency resolution
7. **Document all versions found** — Users need to know which version they're using