# MCP and CLI Debug Checklist

Use this reference when a command, local tool, or MCP server fails to start or execute.

## When to Use

Use this checklist in these scenarios:
- MCP server fails to start or exits immediately
- CLI command not found or throws error
- Skill invocation produces errors
- Tool execution result is unexpected

## Diagnostic Workflow

### Step 1: Reproduce the Problem

Run the minimal reproduction command to confirm the failure layer:

```powershell
# Basic command tests
tool --help
npm run dev -- --help
node dist/index.js --help
python -m package_name --help
paper-fetch --help
paper-fetch-mcp
mineru --help
mmdc -h

# MCP server tests
npx mcp-server --help
node mcp-server.js 2>&1
```

### Step 2: Extract MCP Configuration

For MCP issues, extract the exact host configuration:

```text
command: [Command]
args: [Arguments array]
cwd: [Working directory]
env: [Environment variables]
transport: [stdio/sse]
server logs: [Log output]
host error message: [Host error message]
```

### Step 3: Failure Classification

Classify failure type based on symptoms:

| Classification | Symptoms | Detection Method |
| -------------- | -------- | ----------------- |
| missing dependency | ImportError, ModuleNotFoundError | pip list, npm list |
| wrong Python environment | Version conflict, package not found | python --version, which python |
| wrong Node version | engine errors | node --version, npm --version |
| wrong working directory | File not found | pwd, retry after cd |
| wrong entrypoint | command not found | Check bin/ or package.json |
| missing environment variable | KeyError, config missing | echo $VAR |
| Windows path quoting issue | Path with spaces error | Use absolute path and quotes |
| stdio polluted by logs | MCP protocol parsing error | Redirect logs to stderr |
| process exits immediately | Server starts then immediately exits | Check log output |
| API key missing | Authentication failed | Check env or config |
| permission issue | Access denied | Check file permissions |
| file path access issue | FileNotFoundError | Check if path exists |
| port conflict | EADDRINUSE | netstat -ano |
| MCP config command/args mismatch | Startup parameter error | Compare mcp.json with actual command |

## Fix Patterns

### Pattern 1: Missing Dependency

```powershell
# Check for missing packages
pip list | grep -i <package>
npm list <package>

# Install missing dependencies
pip install <package>
npm install <package>
```

### Pattern 2: Path Issues (Windows)

```powershell
# Windows absolute paths (use double backslash in JSON)
"C:\\Users\\pc\\project\\bin\\tool.exe"

# Paths with spaces need quotes
"C:\\Program Files\\tool.exe"

# Use absolute paths in MCP config cwd
"cwd": "C:\\Users\\pc\\project"
```

### Pattern 3: Log Pollution of stdio

For stdio MCP, logs must write to stderr, not stdout:

```javascript
// WRONG: console.log writes to stdout, pollutes MCP protocol
console.log("Server started");  
console.log("Processing...");  // This breaks MCP communication

// CORRECT: write to stderr
console.error("Server started");  
console.error("Processing...");  // This does not affect stdio

// For Node.js, also consider:
console.warn("Warning message");  // Also goes to stderr
```

For Python MCP servers:

```python
# WRONG: print() goes to stdout, pollutes MCP protocol
print("Server started")
print("Processing...")

# CORRECT: use sys.stderr
import sys
sys.stderr.write("Server started\n")
sys.stderr.flush()
```

### Pattern 4: Wrong Working Directory

```powershell
# Specify cwd in MCP config (use absolute path)
{
  "command": "node",
  "args": ["dist/server.js"],
  "cwd": "C:\\Users\\pc\\project"
}
```

### Pattern 5: Missing Environment Variables

```powershell
# Check required environment variables (Windows)
echo %API_KEY%
echo %DATABASE_URL%

# Check (Unix)
echo $API_KEY

# Pass in MCP config
{
  "env": {
    "API_KEY": "your-key-here",
    "DATABASE_URL": "postgres://localhost:5432/db"
  }
}
```

### Pattern 6: Python Environment Issues

```powershell
# Check Python path (Windows)
where python
py -0  # List all installed versions

# Specify interpreter explicitly
"C:\\Python312\\python.exe"

# Virtual environment (Windows)
"C:\\Users\\pc\\venv\\Scripts\\python.exe"
```

### Pattern 7: Node.js Version Issues

```powershell
# Check Node version
node --version
npm --version

# Check package.json engines requirement
# Compare with actual version

# Use nvm to switch versions (if available)
nvm use 18
```

## Verification Format

After fixing, return verification report in this format:

```text
根因：[One-sentence description]
失败层级：[Classification]
修复方案：[Specific change]
验证命令：[Executable command]
实际输出：[Command output]
残余风险：[Possible remaining issues]
```

## Verification Rules

1. **Must run verification commands** — Cannot rely on speculation
2. **Record actual output** — Prove the fix is effective
3. **Explain unverifiable situations** — Such as requiring network or remote resources
4. **Don't claim fix success** — Unless verification command passes

## Windows-Specific Rules

1. **Use absolute paths in MCP config** — Relative paths are resolved differently
2. **Wrap paths with spaces in quotes** — `C:\Program Files\nodejs\node.exe`
3. **Separate command and args** — Don't put complete shell command in command field
   - WRONG: `"command": "node dist/server.js --port 3000"`
   - CORRECT: `"command": "node", "args": ["dist/server.js", "--port", "3000"]`
4. **Use cwd instead of fragile relative paths** — Always set working directory explicitly
5. **Record Python tool interpreter path** — Not just `python`, use `C:\Python312\python.exe`
6. **Use full path for Node.js tools** — Like `C:\Program Files\nodejs\node.exe`
7. **Use double backslash in JSON config** — `C:\\Users\\pc\\project`

## Diagnostic Checklist

- [ ] Run minimal reproduction command
- [ ] Extract MCP configuration (command/args/cwd/env/transport)
- [ ] Classify failure type
- [ ] Apply minimal fix
- [ ] Run verification command
- [ ] Record actual output
- [ ] Explain remaining risks

## Common Fix Scenarios

### Scenario: MCP Server Exits Immediately

```
症状：MCP 服务器启动后立即退出，无错误信息
排查：
1. 检查日志输出（可能是日志写到 stdout 污染协议）
2. 检查 cwd 是否正确
3. 检查 command/args 是否匹配
4. 检查是否有依赖缺失

解决：
1. 将 console.log 改为 console.error
2. 确认 cwd 使用绝对路径
3. 对比 package.json bin 和实际命令
4. 运行 pip list / npm list 检查依赖
```

### Scenario: CLI Command Not Found

```
症状：command not found 或 'xxx' is not recognized
排查：
1. 检查命令是否安装
2. 检查 PATH 环境变量
3. 检查 bin/ 目录是否存在
4. 检查 package.json bin 字段

解决：
1. 使用完整路径代替命令名
2. 将路径添加到 PATH
3. 检查 npm link 或 pip install
```

### Scenario: Python Package Not Found

```
症状：ModuleNotFoundError 或 ImportError
排查：
1. 检查 python 版本
2. 检查 pip 安装的包
3. 检查虚拟环境

解决：
1. 指定正确的 Python 解释器
2. 安装缺失的包
3. 激活正确的虚拟环境
```