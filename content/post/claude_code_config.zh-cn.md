---
title: "我的 Claude Code 配置实践"
date: 2026-02-23T20:00:00+08:00
draft: false
tags: [claude-code, mcp, ai]
categories: [tech]
---

2025 年 2 月，[Claude Code](https://claude.com/product/claude-code) 发布，带来了全新的 **agentic coding** 体验。经过一年的发展，Claude Code 已经成为我日常开发中不可或缺的工具。本文分享我的 Claude Code 配置和实践经验，希望能给正在使用或准备使用 Claude Code 的开发者一些参考。

## 什么是 MCP

在介绍配置之前，先简单解释一下 **MCP (Model Context Protocol)**。这是 Anthropic 提出的开放协议，用于标准化 AI 助手与外部工具、数据源的连接方式。你可以把 MCP 理解为 AI 的"插件系统"——通过 MCP，Claude Code 可以访问数据库、操作 GitHub、控制浏览器等，极大扩展了 AI 的能力边界。

MCP 服务器有两种连接方式：

- **HTTP**：远程服务，通过 URL 连接，适合访问云端 API
- **stdio**：本地进程，通过标准输入输出通信，适合本地工具

## 我的 MCP 配置

我目前配置了 4 个 MCP 服务器，涵盖代码查询、代码托管、浏览器调试和网页获取四个维度。

### context7 - 代码文档查询

```json
{
  "context7": {
    "type": "http",
    "url": "https://mcp.context7.com/mcp",
    "headers": {
      "CONTEXT7_API_KEY": "YOUR_CONTEXT7_API_KEY"
    }
  }
}
```

**作用**：查询流行编程库的官方文档和代码示例。

**使用场景**：
- 查询某个库的 API 用法，比如 "Next.js 的 Image 组件有哪些参数"
- 获取代码示例，了解最佳实践
- 对比不同版本的 API 差异

**获取 API Key**：访问 [Context7 官网](https://context7.com/) 注册获取。

### github - GitHub 集成

```json
{
  "github": {
    "type": "http",
    "url": "https://api.githubcopilot.com/mcp",
    "headers": {
      "Authorization": "Bearer YOUR_GITHUB_TOKEN"
    }
  }
}
```

**作用**：直接在 Claude Code 中管理 GitHub 仓库、PR 和 Issues。

**使用场景**：
- 创建 PR 并添加详细描述
- 审查代码并添加评论
- 管理 Issues（创建、更新、关闭）
- 查询仓库文件内容

**获取 Token**：在 GitHub Settings → Developer settings → Personal access tokens 中创建，需要 `repo` 和 `read:org` 权限。

### chrome-devtools - 浏览器调试

```json
{
  "chrome-devtools": {
    "type": "stdio",
    "command": "npx",
    "args": [
      "chrome-devtools-mcp@latest"
    ],
    "env": {}
  }
}
```

**作用**：连接 Chrome 开发者工具，进行页面调试、截图、性能分析。

**使用场景**：
- 前端开发时实时查看页面效果
- 自动生成页面截图
- 进行性能分析，优化加载速度
- 监控网络请求

**前提条件**：需要安装 Chrome 浏览器，并保持 DevTools 协议端口开放。

### server-fetch - 网页获取

```json
{
  "server-fetch": {
    "command": "uvx",
    "args": [
      "mcp-server-fetch"
    ]
  }
}
```

**作用**：抓取网页内容并提取为 Markdown。

**使用场景**：
- 获取技术文档内容作为上下文
- 抓取 API 参考页面
- 读取在线教程和博客文章

**注意**：使用 `uvx` 运行，需要提前安装 [uv](https://github.com/astral-sh/uv)（Python 包管理器）。

## 推荐但暂未配置的 MCP

以下 MCP 服务器也很有价值，根据你的需求选择性配置：

| MCP 服务器 | 用途 | 适用场景 |
|-----------|------|---------|
| **Linear** / **Asana** | 项目管理 | 将 AI 工作流与任务管理集成 |
| **Slack** | 团队通知 | 让 Claude Code 发送进度通知 |
| **Supabase** / **Firebase** | 数据库操作 | 直接查询和修改数据库 |
| **Stripe** | 支付集成 | 测试支付流程、查询交易记录 |
| **Playwright** | 浏览器自动化 | 端到端测试、自动化操作 |

## Skill 配置

除了 MCP，Claude Code 还支持 **Skill**（技能），用于执行特定的高级任务。

根据我的使用记录，以下 Skill 使用频率较高：

| Skill | 使用次数 | 用途 |
|-------|---------|------|
| **drawio** | 3 | 生成 draw.io 图表，用于架构图、流程图 |
| **frontend-design** | 1 | 创建高质量前端界面 |
| **claude-hud:setup** | 1 | 配置 HUD 状态栏 |

**drawio** 是我使用最多的 Skill，在讨论系统架构时特别有用——直接生成可编辑的图表文件，比文字描述直观得多。

## 插件配置

我还安装了两个插件来增强体验：

### Claude HUD

[claude-hud](https://github.com/4lch4/claude-hud) 是一个状态栏 HUD 插件，在终端底部显示：

- 当前使用的工具
- Agent 活动状态
- 待办事项列表
- 会话时长

安装命令：
```bash
claude config plugins add claude-hud@claude-hud
```

### Frontend Design

[frontend-design](https://github.com/claude-plugins/frontend-design) 是官方插件，用于创建高质量的前端界面。适合快速生成原型代码或参考实现。

安装命令：
```bash
claude config plugins add frontend-design@claude-plugins-official
```

## 配置方式对比与实践心得

### HTTP vs stdio

两种 MCP 配置方式各有优劣：

**HTTP 模式**：
- ✅ 配置简单，只需 URL 和 API Key
- ✅ 服务稳定，无需本地环境
- ❌ 依赖网络，有延迟
- ❌ 需要管理 API Key 安全

**stdio 模式**：
- ✅ 本地运行，延迟低
- ✅ 可以访问本地资源
- ❌ 需要配置运行环境（Node.js/Python）
- ❌ 每次启动需要初始化进程

**我的建议**：
- 外部服务（GitHub、Context7）用 HTTP
- 本地工具（浏览器调试、文件操作）用 stdio

### API Key 管理建议

配置中涉及多个 API Key，安全管理很重要：

1. **不要将 `.claude.json` 提交到 Git**：添加 `.gitignore` 排除
2. **使用环境变量**：敏感信息可以通过环境变量注入，例如：
   ```json
   {
     "headers": {
       "Authorization": "Bearer ${GITHUB_TOKEN}"
     }
   }
   ```
3. **定期轮换 Token**：GitHub Token 支持设置过期时间，建议启用
4. **最小权限原则**：给 Token 分配最小必要的权限范围

### 权限配置注意事项

Claude Code 在执行 MCP 工具时会请求权限，建议：

- **开发初期**：选择 "Ask before each use"，了解工具调用情况
- **熟悉后**：对可信工具选择 "Always allow"，提升效率
- **生产环境**：谨慎授权，避免误操作

## 完整配置参考

这是我的 `~/.claude.json` 中 MCP 相关部分的完整配置（已脱敏）：

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_CONTEXT7_API_KEY"
      }
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_GITHUB_TOKEN"
      }
    },
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest"
      ],
      "env": {}
    },
    "server-fetch": {
      "command": "uvx",
      "args": [
        "mcp-server-fetch"
      ]
    }
  }
}
```

## 总结

这套配置覆盖了我在日常开发中最常用的场景：

- **查文档** → Context7
- **管代码** → GitHub
- **调页面** → Chrome DevTools
- **抓网页** → server-fetch
- **画图表** → drawio Skill

MCP 生态系统正在快速发展，新的服务器每天都在涌现。建议先从核心需求出发，逐步扩展配置，找到最适合自己工作流的组合。

如果你还没有尝试过 Claude Code，现在是个好时机。2026 年的 AI 编程工具已经今非昔比，熟练掌握它们，将极大提升开发效率。
