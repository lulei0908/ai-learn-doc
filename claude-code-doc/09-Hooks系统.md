# 09 - Hooks 系统

## 概述

Hooks 是用户定义的 Shell 命令、HTTP 端点或 LLM 提示，在 Claude Code 生命周期的特定时间点自动执行。用于自动化工作流、扩展权限评估、验证操作等。

## Hook 生命周期

```
SessionStart
    ↓
UserPromptSubmit
    ↓
┌─────────────────────────────────────┐
│         Agentic Loop               │
│  PreToolUse → PermissionRequest →  │
│  Tool Execution → PostToolUse      │
│         (循环执行)                   │
└─────────────────────────────────────┘
    ↓
Stop / StopFailure
    ↓
PreCompact → PostCompact
    ↓
SessionEnd
```

## Hook 事件

| 事件 | 触发时机 | 可阻塞？ |
|------|---------|---------|
| `SessionStart` | 会话开始或恢复 | 否 |
| `UserPromptSubmit` | 提交提示后，Claude 处理前 | 是 |
| `PreToolUse` | 工具调用执行前 | 是 |
| `PermissionRequest` | 权限对话框出现时 | 是 |
| `PostToolUse` | 工具调用成功后 | 否 |
| `PostToolUseFailure` | 工具调用失败后 | 否 |
| `SubagentStart` | 子代理启动时 | 否 |
| `SubagentStop` | 子代理完成时 | 是 |
| `TaskCreated` | 任务创建时 | 是 |
| `TaskCompleted` | 任务标记完成时 | 是 |
| `Stop` | Claude 完成响应时 | 是 |
| `StopFailure` | API 错误导致停止时 | 否 |
| `TeammateIdle` | 代理团队成员即将空闲时 | 是 |
| `InstructionsLoaded` | CLAUDE.md 文件加载时 | 否 |
| `ConfigChange` | 配置文件变更时 | 是 |
| `CwdChanged` | 工作目录变更时 | 否 |
| `FileChanged` | 监视的文件变更时 | 否 |
| `WorktreeCreate` | Worktree 创建时 | 是 |
| `WorktreeRemove` | Worktree 移除时 | 否 |
| `PreCompact` | 上下文压缩前 | 否 |
| `PostCompact` | 上下文压缩后 | 否 |
| `Elicitation` | MCP 请求用户输入时 | 是 |
| `ElicitationResult` | 用户响应 MCP 输入请求后 | 是 |
| `SessionEnd` | 会话终止时 | 否 |

## 配置位置

| 位置 | 作用域 | 可共享 |
|------|--------|--------|
| `~/.claude/settings.json` | 所有项目 | 否 |
| `.claude/settings.json` | 单个项目 | 是 |
| `.claude/settings.local.json` | 单个项目 | 否 |
| 托管策略设置 | 企业范围 | 是 |
| 插件 `hooks/hooks.json` | 插件启用范围 | 是 |
| 技能/代理 frontmatter | 组件激活期间 | 是 |

## 配置结构

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./validate.sh"
          }
        ]
      }
    ]
  }
}
```

三层结构：
1. **Hook 事件** - 响应的时间点
2. **Matcher 组** - 过滤触发条件
3. **Hook 处理器** - 执行的操作

## Matcher 模式

| 事件 | Matcher 过滤字段 | 示例 |
|------|-----------------|------|
| `PreToolUse`, `PostToolUse` | 工具名称 | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | 会话启动方式 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | 会话结束原因 | `clear`, `resume`, `logout` |
| `Notification` | 通知类型 | `permission_prompt`, `idle_prompt` |
| `SubagentStart/Stop` | 代理类型 | `Explore`, `Plan`, 自定义名称 |
| `FileChanged` | 文件名 | `.envrc`, `.env` |
| `ConfigChange` | 配置来源 | `user_settings`, `project_settings` |

## Hook 处理器类型

### 命令 Hook

```json
{
  "type": "command",
  "command": "./scripts/validate.sh",
  "timeout": 60,
  "async": false,
  "shell": "bash"
}
```

### HTTP Hook

```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/pre-tool-use",
  "timeout": 30,
  "headers": {
    "Authorization": "Bearer $MY_TOKEN"
  },
  "allowedEnvVars": ["MY_TOKEN"]
}
```

### Prompt Hook

```json
{
  "type": "prompt",
  "prompt": "分析这个操作是否安全：$ARGUMENTS",
  "model": "haiku",
  "timeout": 30
}
```

### Agent Hook

```json
{
  "type": "agent",
  "prompt": "验证这个操作：$ARGUMENTS",
  "model": "sonnet",
  "timeout": 60
}
```

## 公共字段

| 字段 | 说明 |
|------|------|
| `type` | 处理器类型（command/http/prompt/agent） |
| `if` | 权限规则语法过滤 |
| `timeout` | 超时秒数 |
| `statusMessage` | 运行时显示的旋转消息 |
| `once` | 是否只运行一次（技能专用） |

## 命令 Hook 字段

| 字段 | 说明 |
|------|------|
| `command` | Shell 命令 |
| `async` | 是否后台运行 |
| `shell` | Shell 类型（bash/powershell） |

## 退出码行为

| 退出码 | 说明 |
|--------|------|
| 0 | 成功，解析 stdout 的 JSON |
| 2 | 阻塞错误，stderr 反馈给 Claude |
| 其他 | 非阻塞错误，继续执行 |

### 退出码 2 效果

| 事件 | 效果 |
|------|------|
| `PreToolUse` | 阻塞工具调用 |
| `PermissionRequest` | 拒绝权限 |
| `UserPromptSubmit` | 阻塞提示处理 |
| `Stop` | 阻止停止，继续对话 |
| `PostToolUse` | 显示错误（工具已执行） |

## JSON 输入

Hook 通过 stdin 接收 JSON 输入：

```json
{
  "session_id": "abc123",
  "transcript_path": "/home/user/.claude/projects/.../transcript.jsonl",
  "cwd": "/home/user/my-project",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

### 公共字段

| 字段 | 说明 |
|------|------|
| `session_id` | 会话 ID |
| `transcript_path` | 对话记录路径 |
| `cwd` | 当前工作目录 |
| `permission_mode` | 当前权限模式 |
| `hook_event_name` | 事件名称 |
| `agent_id` | 子代理 ID（仅子代理内） |
| `agent_type` | 代理类型（仅代理内） |

## JSON 输出

Exit 0 时解析 stdout 的 JSON：

### 通用字段

```json
{
  "continue": true,
  "stopReason": "构建失败，修复错误后继续",
  "suppressOutput": false,
  "systemMessage": "警告信息"
}
```

### PreToolUse 决策

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "阻塞原因"
  }
}
```

`permissionDecision` 值：
- `allow` - 允许
- `deny` - 拒绝
- `ask` - 提示用户

### PermissionRequest 决策

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "deny"
    }
  }
}
```

## 示例配置

### 阻塞危险命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Hook 阻塞了危险命令"
    }
  }'
else
  exit 0
fi
```

### 自动 Lint

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint --fix"
          }
        ]
      }
    ]
  }
}
```

### 会话开始通知

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '会话开始于: '$(date)"
          }
        ]
      }
    ]
  }
}
```

### 环境变量持久化

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "[ -f .env ] && export $(cat .env | xargs) && env"
          }
        ]
      }
    ]
  }
}
```

## HTTP Hook 示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/pre-tool-use",
            "timeout": 30,
            "headers": {
              "Authorization": "Bearer $MY_TOKEN"
            },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

## 技能和代理中的 Hooks

```yaml
---
name: secure-operations
description: 带安全检查的操作
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

## 查看 Hooks

```
/hooks
```

显示所有配置的 hooks，包括来源：
- User: `~/.claude/settings.json`
- Project: `.claude/settings.json`
- Local: `.claude/settings.local.json`
- Plugin: 插件 `hooks/hooks.json`
- Session: 当前会话内存中

## 禁用 Hooks

```json
{
  "disableAllHooks": true
}
```

**注意**：托管设置的 hooks 不能被用户设置禁用。

## 下一步

- 🔐 配置 [权限](./10-权限配置.md)
- ⚙️ 设置 [模型配置](./11-模型配置.md)
