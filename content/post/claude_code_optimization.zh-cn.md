---
title: "优化 Claude Code Context 使用：轻量级工具替代方案"
date: 2026-03-14T12:00:00+08:00
draft: false
tags: [claude-code, mcp, skill, ai, optimization]
categories: [tech]
---
![title](https://cdn.mahaoliang.tech/2026/20260314120538228.png)

在上个月的文章《我的 Claude Code 配置实践》中，我分享了自己配置 MCP 服务器的经验。但经过这段时间的深度使用，我发现了一个关键问题：**MCP 服务器非常占用 context**。在调研官方文档和工程实践后，我决定将部分 MCP 配置迁移到更轻量级的 Skill 和 CLI 工具上。本文记录这次优化的过程和心得。

## 问题分析：MCP 的 Context 成本

根据 [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/features-overview)，**MCP 服务器在每个请求中都会加载其工具定义和 JSON 架构**。这意味着即使你没有调用某个 MCP 工具，它仍然会消耗宝贵的上下文窗口。

| 扩展类型 | 典型 Token 消耗 | 加载方式 |
|---------|---------------|---------|
| MCP 服务器 | 8,000 - 50,000+ | 每个请求自动加载 |
| Skill | 500 - 2,000 | 按需调用 |
| CLI 工具 | 0（不占用 context） | 通过 Bash 调用 |

当你配置了多个 MCP 服务器时，这个问题会更加明显。比如同时启用 `chrome-devtools` 和 `github` 两个 MCP，每次请求可能就要消耗数万个 token 的上下文空间，留给实际代码分析和对话的空间就被压缩了。

## 方案一：chrome-devtools MCP → agent-browser Skill

### 为什么选择迁移

| 对比维度 | agent-browser Skill | Chrome DevTools MCP |
|---------|---------------------|---------------------|
| **Token 消耗** | 减少 **93%** (500-800 vs 8,000-50,000+) | 高 |
| **视觉回归测试** | 支持快照对比、像素级差异检测 | 不支持 |
| **网络请求拦截** | 支持拦截、模拟、阻塞 | 仅监控 |
| **安装方式** | `npx skills add vercel-labs/agent-browser` | JSON 配置 |

### 安装与配置

```bash
# 安装 agent-browser Skill
npx skills add vercel-labs/agent-browser
```

安装完成后，Claude Code 会自动识别这个 Skill，无需在配置文件中手动添加。

### 使用对比

**Chrome DevTools MCP 的方式：**

```
# 需要 MCP 服务器在后台运行
# 通过复杂的 JSON-RPC 调用
# Token 消耗大
```

**agent-browser Skill 的方式：**

```
# 直接使用自然语言指令
"打开 https://example.com 并截图"
"点击登录按钮"
"截取当前页面的视觉快照用于对比"
```

### 功能增益

除了更低的 context 成本，`agent-browser` 还带来了额外能力：

1. **视觉回归测试**：可以对比两个版本的页面截图，检测像素级差异
2. **网络请求拦截**：可以拦截、修改或模拟网络请求，而不仅仅是监控
3. **更适合 AI 的 API**：针对 AI agent 的使用场景设计，操作更直观

## 方案二：GitHub MCP → gh CLI

### 为什么选择迁移

| 对比维度 | gh CLI | GitHub MCP |
|---------|--------|-----------|
| **Token 效率** | ~1,365 tokens（便宜 10-32 倍） | ~44,026 tokens |
| **可靠性** | 100%（本地运行） | 72%（连接超时问题） |
| **成本** | ~$3.20/月（10,000 次操作） | ~$55.20/月 |
| **安装方式** | `brew install gh && gh auth login` | JSON 配置 + Token |

### 安装与配置

```bash
# 安装 GitHub CLI
brew install gh

# 认证登录
gh auth login

# 按照提示选择：
# - GitHub.com
# - HTTPS
# - 使用浏览器登录（推荐）或 Token 登录
```

### 常用命令对照表

| 操作 | GitHub MCP | gh CLI |
|-----|-----------|--------|
| 查看 PR 列表 | MCP 调用 | `gh pr list` |
| 查看 PR 详情 | MCP 调用 | `gh pr view 123` |
| 创建 PR | MCP 调用 | `gh pr create` |
| 合并 PR | MCP 调用 | `gh pr merge 123` |
| 查看 Issues | MCP 调用 | `gh issue list` |
| Fork 仓库 | MCP 调用 | `gh repo fork` |
| Clone 仓库 | MCP 调用 | `gh repo clone owner/repo` |

### 在 Claude Code 中使用 gh 的最佳实践

**直接调用：**

```
用户：查看一下我的 open PR
Claude：我来帮你查看当前的 PR 列表。
```

Claude 会直接执行：
```bash
gh pr list
```

**结合其他命令：**

```
用户：创建一个新的 PR，标题是"修复登录问题"
Claude：我来帮你创建 PR。
```

Claude 会执行：
```bash
gh pr create --title "修复登录问题" --body "$(cat <<'EOF'
## 修复内容
- 修复登录验证逻辑
- 更新错误提示信息
EOF
)"
```

## 配置对比

### 优化前（部分 MCP 配置）

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxxx"
      }
    },
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-chrome-devtools"]
    }
  }
}
```

### 优化后

```json
{
  "mcpServers": {
    // chrome-devtools MCP 已移除
    // github MCP 已移除
    // 其他保留的 MCP...
  }
}
```

- `agent-browser`：通过 `npx skills add` 安装，不占用 MCP 配置
- `gh`：通过 `brew install gh` 安装，通过 Bash 调用

## 实践心得

### 什么场景适合保留 MCP

1. **需要结构化数据返回**：MCP 返回的是结构化数据，Claude 可以更好地理解和处理
2. **需要与 IDE 深度集成**：比如 context7 查询文档，返回的内容直接用于代码补全
3. **复杂的多步骤操作**：MCP 可以维护状态，适合复杂的工具链调用

### 什么场景适合使用 Skill

1. **浏览器自动化**：`agent-browser` 专为 AI 设计，功能更强、成本更低
2. **特定领域的专业工具**：Skill 通常针对特定场景优化
3. **需要视觉/截图能力**：Skill 在这方面有明显优势

### 什么场景适合使用 CLI

1. **简单直接的操作**：如 `gh pr list`、`git status` 等单次调用
2. **成本敏感的场景**：CLI 不占用 context，成本最低
3. **可靠性要求高的场景**：本地 CLI 不会遇到 MCP 连接超时问题

### 混合使用策略

我的最终配置策略：

| 工具类型 | 保留/新增 | 用途 |
|---------|----------|------|
| context7 MCP | 保留 | 文档查询，需要结构化数据 |
| agent-browser Skill | 新增 | 浏览器自动化 |
| gh CLI | 新增 | GitHub 操作 |
| chrome-devtools MCP | 移除 | 被 agent-browser 替代 |
| github MCP | 移除 | 被 gh CLI 替代 |

## 总结

通过这次优化，我获得了显著的改进：

- **Token 消耗**：预计减少 50-80%，尤其是浏览器和 GitHub 相关操作
- **响应速度**：减少了 MCP 加载时间，Claude 的响应更快
- **可靠性提升**：避免了 MCP 连接失败的问题
- **成本降低**：如果使用 API 调用，月度成本从 $50+ 降低到 $5 以下

更重要的是，这次优化让我重新思考了工具选择的原则：**不是功能越多越好，而是要在功能、成本和可靠性之间找到平衡**。MCP 是一个优秀的协议，但并不意味着所有操作都要通过 MCP 完成。Skill 和 CLI 在很多场景下是更轻量、更高效的选择。

未来我计划继续关注 Claude Code 生态的发展，探索更多轻量级工具，同时保持配置的简洁和高效。

---

**参考链接：**

- [agent-browser 文档](https://agent-browser.dev)
- [GitHub CLI 文档](https://cli.github.com)
- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/features-overview)
