# Claude Code 完整使用文档

> 本文档全面介绍 Anthropic Claude Code 的安装、配置和使用方法。Claude Code 是 Anthropic 官方推出的 AI 编程助手 CLI 工具。

## 📚 文档目录

### 入门指南
- [01-快速开始](./01-快速开始.md) - 安装、基础使用和第一个项目
- [02-CLI命令参考](./02-CLI命令参考.md) - 所有命令行参数和选项
- [03-交互模式](./03-交互模式.md) - 会话内命令和快捷键

### 核心功能
- [04-工具参考](./04-工具参考.md) - Claude Code 可用的所有工具
- [05-记忆系统](./05-记忆系统.md) - CLAUDE.md 配置和上下文管理
- [06-技能系统](./06-技能系统.md) - 扩展 Claude 能力的自定义技能
- [07-子代理](./07-子代理.md) - 专业化的 AI 子代理配置

### 集成与扩展
- [08-MCP集成](./08-MCP集成.md) - Model Context Protocol 工具集成
- [09-Hooks系统](./09-Hooks系统.md) - 自动化工作流和生命周期钩子
- [10-权限配置](./10-权限配置.md) - 细粒度权限控制
- [11-模型配置](./11-模型配置.md) - 模型选择和配置

### 最佳实践
- [12-常见工作流](./12-常见工作流.md) - 高效使用 Claude Code 的模式
- [13-最佳实践](./13-最佳实践.md) - 推荐的使用方式和配置

---

## 什么是 Claude Code？

Claude Code 是 Anthropic 推出的 AI 编程助手，具有以下特点：

### 核心能力
- **代码理解与生成** - 理解复杂代码库，生成高质量代码
- **终端操作** - 执行 shell 命令，运行测试和构建
- **文件操作** - 读取、编辑、创建文件
- **网络访问** - 搜索网络、获取文档
- **MCP 集成** - 连接数百种外部工具和服务

### 架构特点
- **代理式执行** - Claude 自主规划和执行任务
- **上下文感知** - 理解项目结构和代码关系
- **工具调用** - 通过工具系统与外部世界交互
- **可扩展性** - 通过 Skills、Hooks、MCP 扩展能力

## 快速安装

```bash
# 使用 npm 安装
npm install -g @anthropic-ai/claude-code

# 或使用 npx 直接运行
npx @anthropic-ai/claude-code

# 验证安装
claude --version
```

## 基本使用

```bash
# 启动交互式会话
claude

# 执行单个任务
claude "解释这个项目的架构"

# 指定工作目录
claude --cd /path/to/project

# 使用特定模型
claude --model opus
```

## 配置层级

Claude Code 使用分层配置系统：

```
优先级（从高到低）：
1. 托管设置（Managed Settings）    - 企业级策略
2. 命令行参数                      - 临时覆盖
3. 本地项目设置                    - .claude/settings.local.json
4. 共享项目设置                    - .claude/settings.json
5. 用户设置                        - ~/.claude/settings.json
```

## 支持的模型

| 别名 | 说明 |
|------|------|
| `default` | 推荐模型（根据账户类型自动选择） |
| `sonnet` | Claude Sonnet 4.6（日常编程） |
| `opus` | Claude Opus 4.6（复杂推理） |
| `haiku` | Claude Haiku（快速简单任务） |
| `sonnet[1m]` | Sonnet 100万 token 上下文 |
| `opus[1m]` | Opus 100万 token 上下文 |
| `opusplan` | 计划模式用 Opus，执行用 Sonnet |

## 相关资源

- 📖 [官方文档](https://docs.anthropic.com/en/docs/claude-code)
- 🐙 [GitHub 仓库](https://github.com/anthropics/claude-code)
- 💬 [Discord 社区](https://discord.com/invite/anthropic)
- 🔌 [MCP 服务器市场](https://github.com/modelcontextprotocol/servers)

---

*文档版本：1.0 | 最后更新：2026-03-29*
