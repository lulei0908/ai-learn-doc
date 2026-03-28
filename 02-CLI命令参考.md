# 02 - CLI 命令参考

## 基本语法

```bash
claude [options] [prompt]
```

## 全局选项

### 认证与账户

| 选项 | 说明 |
|------|------|
| `--auth-key <key>` | 使用 API 密钥认证 |
| `--no-input` | 无头模式，不等待输入 |

### 模型选择

| 选项 | 说明 |
|------|------|
| `--model <model>` | 选择模型（sonnet, opus, haiku, opusplan） |
| `--effort <level>` | 设置努力级别（low, medium, high, max） |

### 会话管理

| 选项 | 说明 |
|------|------|
| `--continue` | 继续上次会话 |
| `--resume` | 从会话列表选择恢复 |
| `--new` | 启动全新会话 |
| `--session <id>` | 恢复指定会话 |
| `--clear` | 清除当前会话并重新开始 |

### 工作目录

| 选项 | 说明 |
|------|------|
| `--cd <path>` | 指定工作目录 |
| `--add-dir <path>` | 添加额外可访问目录 |

### 调试与日志

| 选项 | 说明 |
|------|------|
| `--debug` | 启用调试模式 |
| `--verbose` | 输出详细日志 |
| `--output <file>` | 保存会话输出到文件 |

### 代理与网络

| 选项 | 说明 |
|------|------|
| `--max-budget <dollars>` | 设置最大花费预算 |
| `--print <format>` | 输出格式（text, pretty, stream-json） |

### MCP 配置

| 选项 | 说明 |
|------|------|
| `--mcp <config>` | 指定 MCP 配置文件 |
| `--mcp-tool-search` | 启用 MCP 工具搜索 |

### 帮助

```bash
claude --help          # 显示帮助信息
claude help <topic>    # 显示特定主题帮助
```

## 子命令

### claude agents

管理自定义子代理。

```bash
# 列出所有子代理
claude agents

# 创建新子代理（交互式）
claude agents create
```

### claude mcp

管理 MCP 服务器。

```bash
# 列出所有已配置的 MCP 服务器
claude mcp list

# 添加新的 MCP 服务器
claude mcp add <name> -- <command>
claude mcp add --transport http <name> <url>
claude mcp add --transport stdio <name> -- <command>

# 移除 MCP 服务器
claude mcp remove <name>

# 查看服务器详情
claude mcp get <name>

# 从 Claude Desktop 导入
claude mcp add-from-claude-desktop
```

#### MCP 添加示例

```bash
# HTTP 服务器
claude mcp add github --transport http https://api.githubcopilot.com/mcp/

# 带认证的 HTTP 服务器
claude mcp add sentry --transport http https://mcp.sentry.dev/mcp \
  --header "Authorization: Bearer $SENTRY_TOKEN"

# 本地 stdio 服务器
claude mcp add filesystem --transport stdio -- npx -y @modelcontextprotocol/server-filesystem .

# 带环境变量
claude mcp add db --transport stdio --env DATABASE_URL=xxx -- npx -y @bytebase/dbhub \
  --dsn "$DATABASE_URL"
```

### claude auto-mode

管理 auto 模式配置。

```bash
# 显示默认配置
claude auto-mode defaults

# 显示当前生效配置
claude auto-mode config

# 获取 AI 对自定义规则的反馈
claude auto-mode critique
```

### claude plugins

管理插件。

```bash
# 列出可用插件
claude plugins list

# 安装插件
claude plugins add <name>

# 卸载插件
claude plugins remove <name>

# 查看插件详情
claude plugins info <name>
```

### claude auth

认证管理。

```bash
# 登录
claude auth login

# 登出
claude auth logout

# 状态
claude auth status
```

### claude init

在当前目录初始化 Claude Code 配置。

```bash
claude init
```

## 环境变量

### Anthropic API 配置

```bash
export ANTHROPIC_API_KEY=sk-ant-api03-xxxxx
export ANTHROPIC_BASE_URL=https://api.anthropic.com  # 可选，默认值
```

### 模型配置

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-6
export CLAUDE_CODE_SUBAGENT_MODEL=sonnet
```

### MCP 配置

```bash
export ENABLE_TOOL_SEARCH=true          # 启用 MCP 工具搜索
export MCP_TIMEOUT=10000                 # MCP 服务器超时（毫秒）
export MAX_MCP_OUTPUT_TOKENS=50000      # MCP 工具输出最大 token 数
```

### 会话配置

```bash
export CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1  # 保持工作目录
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1      # 禁用后台任务
export CLAUDE_CODE_DISABLE_1M_CONTEXT=1            # 禁用 100万上下文
```

### 调试配置

```bash
export CLAUDE_CODE_DEBUG=1
export CLAUDE_CODE_VERBOSE=1
```

## 退出码

| 退出码 | 说明 |
|--------|------|
| 0 | 成功完成 |
| 1 | 一般错误 |
| 130 | 被 Ctrl+C 中断 |

## 命令行使用示例

### 基础使用

```bash
# 标准会话
claude

# 单次任务
claude "解释这个函数的作用"

# 指定目录
claude --cd /path/to/project "分析代码库"

# 继续上次会话
claude --continue
```

### 高级使用

```bash
# 使用特定模型
claude --model opus "设计微服务架构"

# 启用调试
claude --debug "运行并分析这个脚本"

# 添加额外目录
claude --add-dir /shared/lib "使用共享库"

# 保存输出
claude --output session.log "分析这个bug"

# 管道输入
cat error.log | claude
echo "task" | claude --no-input
```

### MCP 使用

```bash
# 添加 MCP 服务器
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/projects

# 添加带认证的服务器
claude mcp add github --transport http https://api.githubcopilot.com/mcp/
```
