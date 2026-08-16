---
title: "本机 Skills 的安装和管理"
date: 2026-06-16T09:33:22+08:00
draft: false
tags: [ai, agent, claude-code, skill]
categories: [tech]
---

skill 给 agent 赋予了各种经验和能力。随着使用的深入，本机已经安装了不少 skills，同时，我从原先只使用 Claude code，到现在要使用 codex、Kimi cli 等多个 agent，需要对 skills 进行整理，有些可以多 agent 共享使用，有些则只能针对特定 agent 安装。

## 多 Agent 共享使用

最初所有 skill 都是在 Claude code 里安装，后来发现可以使用 `npx skills` 进行统一管理。`npx skills`会将 skills 安装在 `${HOME}/.agents/skills`目录下，然后 agent 通过建立符号链接，访问这些 skills。

skills.sh 本质上是一个「中心化存储 + 符号链接分发」的轻量级包管理器。它在 `~/.agents/skills/` 维护一份唯一的物理副本，安装时自动检测本机已安装的 Agent（Claude Code、Cursor、Codex、Trae 等），并在各自的配置目录中创建软链接。更新时只需要 pull 一次中心仓库，所有 Agent 立即同步，不存在版本漂移。目前这个生态已经支持 40 多个 Agent，skill 本身是一个带 `SKILL.md` 的 GitHub 仓库，内容就是程序性知识——告诉 Agent「遇到这个任务时，按什么步骤做」。

![skills.sh](https://cdn.mahaoliang.tech/2026/20260816093719665.svg)

下面是目前我在多个 agent 间共享使用的 skills。上图就是使用  [diagram-design](https://github.com/cathrynlavery/diagram-design) skill 生成的。
### [agent-browser](https://github.com/vercel-labs/agent-browser)

面向 agent 的浏览器自动化命令行，主要用于前端应用测试。

```bash
npx skills add vercel-labs/agent-browser --skill agent-browser -g
```

Codex 有自己的浏览器测试 skill，所以 agent-browser 主要给其他 agent 使用。

### [Kimi WebBridge](https://www.kimi.com/zh-cn/products/kimi-webbridge)

Kimi 发布的浏览器插件 + Skill，可以让 agent 像人类一样与网站交互：搜索、滚动、点击、输入并完成任务。

先安装浏览器插件 [Kimi WebBridge](https://chromewebstore.google.com/detail/kimi-webbridge/fldmhceldgbpfpkbgopacenieobmligc)，然后执行下面的安装脚本：

```bash
curl -fsSL https://cdn.kimi.com/webbridge/install.sh | bash
```

这个脚本会将 skill 安装到以下四个位置：

- `.claude/skills/kimi-webbridge`
- `.codex/skills/kimi-webbridge`
- `.config/agents/skills/kimi-webbridge`
- `.agents/skills/kimi-webbridge`

这样 Claude Code、Codex 和 Kimi Code CLI 等 agent 都能使用。另外，这个安装脚本还会在后台启动一个进程 `.kimi-webbridge/bin/kimi-webbridge`，用来让 skill 操作浏览器。

Codex 有自己的操作 chrome 浏览器的插件 + skill，所以 Kimi WebBridge 主要给 Claude code，Kimi Code CLI 等其他 agent 使用。

### [Mattpocock Skills](https://github.com/mattpocock/skills)

Matt Pocock 的 [skills](https://github.com/mattpocock/skills) 是一套面向 agent 编程的工程方法论，小而精、可组合、基于数十年软件工程经验。


```bash
npx skills@latest add mattpocock/skills -g
```

首次使用时，在目标项目根目录运行 `/setup-matt-pocock-skills`，它会做必要的配置：创建 `docs/agents/` 目录，在 `CLAUDE.md` 或 `AGENTS.md` 中添加 `## Agent skills` 区块。

Matt Pocock 建议的工程流程是：

`grill-with-docs/grill-me → [可选: handoff → prototype → handoff] → to-spec → to-tickets → implement → code-review`


### [diagram-design](https://github.com/cathrynlavery/diagram-design)

生成 27 类型、三种样式的图表。

```bash
npx skills add cathrynlavery/diagram-design -g
```

### [Baoyu Skills](https://github.com/JimLiu/baoyu-skills)

宝玉分享的工具集，我主要使用下面两个：

- 生成文章封面（baoyu-cover-image）
- 下载 YouTube 视频字幕（baoyu-youtube-transcript）

```bash
npx skills add jimliu/baoyu-skills --skill baoyu-cover-image -g
npx skills add jimliu/baoyu-skills --skill baoyu-youtube-transcript -g
```

## Claude code 专有

有些 skill 不仅包含 markdown 和 脚本，还会依赖 agent 的内部机制，例如 Hook。这些 skill 功能很强，需要每个 agent 单独安装。

- [superpowers](https://github.com/obra/superpowers) 开发必备利器
- `commit-commands` ：Claude code 内置的命令。

这两个都在 Claude code，使用官方 marketplace 的搜索安装。

## Codex

Codex 会从 `${HOME}/.agents/skills` 加载 skills，所以我们前面安装的 skills 在 Codex 中都可以使用。下面安装 Codex 特有的插件。

首先安装 [Codex chrome 插件](https://chromewebstore.google.com/detail/codex/hehggadaopoacecdllhhajmbjkdcmajg)，让 Codex 可以控制 Chrome 浏览器。

然后安装 **superpowers 插件**，开发必备。

**Product Design** 插件，根据产品需求，生成高保真原型，利用了 GPT Image 2 的强大能力。

文档处理类插件，**documents**、**Spreadsheets**、**Presentations** 、pdf。

另外 Codex 已经有了 **Browser**（Control the in-app browser with Codex）和 **Chrome: Chrome** 插件，所以我们需要禁用功能重复的 Agent Browser，Kimi WebBridge。

## Kimi Code

根据 Kimi 的[官方文档](https://www.kimi.com/code/docs/kimi-code-cli/customization/skills.html)，Kimi Code 会自动加载 `${HOME}/.agents/skills`下的 skills，所以前面安装共享使用的 skills 在 Kimi Code 中完全能使用。

~~`superpowers` 用到了 agent 的内置功能，目前还不支持 Kimi Code CLI。~~

最新版的 Kimi Code，内置的 [**Marketplace**](https://www.kimi.com/code/docs/kimi-code-cli/customization/plugins.html)，已经提供了 superpowers 插件的安装。

## Kimi 云端 和 桌面 Agent

Kimi 网页版的 Agent 内置了 Kimi 官方的 skill：docx、pdf 和 xlsx，实测比 anthropics 的文档系列 skills 能力强，更加好用。同时，Kimi 网页版的 PPT 功能也很好用。

**文档处理类的需求可以直接交给 Kimi 云端 Agent**。~~如果一定要在本地处理，**建议使用 Codex**，因为它内置了文档类的插件。~~

如果要在本地处理文档，可以使用 Kimi 新出的 [Kimi Work](https://www.kimi.com/zh-cn/products/kimi-work)，和 codex 一样，桌面版 agent。

