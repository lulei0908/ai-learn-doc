# 08 - MCP 集成

## 概述

MCP（Model Context Protocol）是一个开源标准，用于 AI 工具集成。Claude Code 通过 MCP 可以连接数百种外部工具、数据库和 API。

## MCP 能做什么

通过 MCP 服务器，Claude Code 可以：

- **从问题跟踪器实现功能** - "根据 JIRA issue ENG-4521 添加功能并创建 GitHub PR"
- **分析监控数据** - "检查 Sentry 和 Statsig 查看 ENG-4521 描述功能的使用情况"
- **查询数据库** - "基于 PostgreSQL 数据库找出 10 个使用该功能的随机用户邮箱"
- **集成设计工具** - "根据 Slack 中发布的 Figma 设计更新邮件模板"
- **自动化工作流** - "创建 Gmail 草稿邀请这些用户参加反馈会议"
- **响应外部事件** - MCP 服务器可作为渠道推送消息到会话

## 安装 MCP 服务器

### 方式一：HTTP 服务器（推荐）

```bash
# 基本语法
claude mcp add --transport http <name> <url>

# 示例：连接 Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 带 Bearer token
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### 方式二：SSE 服务器（已弃用）

```bash
# 基本语法
claude mcp add --transport sse <name> <url>

# 示例
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

### 方式三：Stdio 服务器（本地）

```bash
# 基本语法
claude mcp add [options] <name> -- <command> [args...]

# 示例：添加 Airtable 服务器
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server

# 添加数据库服务器
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

**重要**：所有选项（`--transport`, `--env`, `--scope`, `--header`）必须在服务器名称**之前**。`--` 分隔服务器名称和要传递给 MCP 服务器的命令。

### 管理服务器

```bash
# 列出所有已配置的服务器
claude mcp list

# 查看服务器详情
claude mcp get github

# 移除服务器
claude mcp remove github

# 会话内检查状态
/mcp
```

## MCP 作用域

### Local 作用域（默认）

存储在 `~/.claude.json`，仅当前项目可用，私人配置。

```bash
claude mcp add --transport http stripe https://mcp.stripe.com
```

### Project 作用域

存储在项目根目录的 `.mcp.json`，可提交版本控制，团队共享。

```bash
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

`.mcp.json` 格式：

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

### User 作用域

存储在 `~/.claude.json`，跨项目可用，个人配置。

```bash
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 作用域选择指南

| 作用域 | 用途 |
|--------|------|
| Local | 个人服务器、实验配置、敏感凭据 |
| Project | 团队共享、项目特定工具 |
| User | 个人工具、跨项目使用 |

## 环境变量展开

在 `.mcp.json` 中支持环境变量展开：

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

**支持语法：**
- `${VAR}` - 展开为变量值
- `${VAR:-default}` - 如果未设置则使用默认值

## OAuth 认证

### 基本流程

```bash
# 1. 添加需要认证的服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 在 Claude Code 中认证
/mcp
# 选择服务器，按照浏览器流程登录
```

### 固定 OAuth 回调端口

```bash
claude mcp add --transport http \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### 预配置 OAuth 凭据

```bash
# 使用客户端 ID 和密钥
claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

## 动态 Headers

对于非 OAuth 认证方案（如 Kerberos、短期 token）：

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-mcp-auth-headers.sh"
    }
  }
}
```

脚本要求：
- 输出 JSON 对象到 stdout
- 10 秒超时
- 动态 headers 覆盖静态 headers

## 从 Claude Desktop 导入

```bash
claude mcp add-from-claude-desktop
# 交互式选择要导入的服务器
```

## 使用 Claude.ai 连接器

如果登录了 Claude.ai 账户，在 Claude.ai 添加的 MCP 服务器自动可用：

1. 在 [claude.ai/settings/connectors](https://claude.ai/settings/connectors) 配置
2. 在 Claude Code 中使用 `/mcp` 查看和管理

禁用：

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

## Claude Code 作为 MCP 服务器

```bash
# 启动为 stdio MCP 服务器
claude mcp serve
```

在 Claude Desktop 中配置：

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "/full/path/to/claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

## MCP 资源引用

使用 `@` 提及引用 MCP 资源：

```
> 分析 @github:issue://123 并建议修复
> 查看 @docs:file://api/authentication 的 API 文档
> 比较 @postgres:schema://users 和 @docs:file://database/user-model
```

## MCP Prompts 作为命令

MCP 服务器可以暴露 prompts，作为命令使用：

```
# 发现可用 prompts
/

# 执行不带参数的 prompt
/mcp__github__list_prs

# 执行带参数的 prompt
/mcp__github__pr_review 456
/mcp__jira__create_issue "登录流程 Bug" high
```

## MCP 工具搜索

默认启用，延迟加载 MCP 工具以减少上下文占用：

- 会话开始时只加载工具名称
- 需要时才加载工具定义
- 实际使用的工具才进入上下文

### 配置

```bash
# 自定义阈值（5%）
ENABLE_TOOL_SEARCH=auto:5 claude

# 禁用工具搜索
ENABLE_TOOL_SEARCH=false claude
```

## MCP 输出限制

- **警告阈值**：10,000 tokens
- **默认上限**：25,000 tokens

调整上限：

```bash
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

## MCP Elicitation（用户输入请求）

MCP 服务器可以请求用户输入：

- **表单模式**：显示带字段的对话框
- **URL 模式**：打开浏览器进行认证或审批

自动处理，无需配置。

## 托管 MCP 配置

### 独占控制

部署 `managed-mcp.json` 到系统目录：

- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux/WSL: `/etc/claude-code/managed-mcp.json`
- Windows: `C:\Program Files\ClaudeCode\managed-mcp.json`

用户无法添加、修改或使用其他服务器。

### 策略控制

使用 allowlist/denylist：

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "sentry" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

## 常用 MCP 服务器示例

### GitHub

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

### Sentry

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

### PostgreSQL 数据库

```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

## 下一步

- 🪝 配置 [Hooks](./09-Hooks系统.md)
- 🔐 设置 [权限](./10-权限配置.md)
