---
title: "打造终端开发环境（2026 agentic coding 版）"
date: 2026-01-13T21:47:28+08:00
draft: false
tags: [macos, terminal, ai]
categories: [tech]
---

2022 年 7 月 23 日，高二暑假，我在自己的第一台 **MacBook Pro（M1）** 上，发布了第一篇博客[《打造一个漂亮的命令行终端》](https://mahaoliang.tech/p/%E6%89%93%E9%80%A0%E4%B8%80%E4%B8%AA%E6%BC%82%E4%BA%AE%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%BB%88%E7%AB%AF/) ，四个月后，也就是 2022 年 11 月 30 日，改变世界的[ChatGPT 发布](https://openai.com/index/chatgpt/)。那时候完全不会想到，AI 的发展竟然对终端的使用产生了极大的影响。那时我使用 macOS 上原生 `terminal`，[Oh My Zsh](https://ohmyz.sh/) + [Powerlevel10k](https://github.com/romkatv/powerlevel10k) 的组合配置，足够漂亮，足够满足我在终端运行 `git` 命令，`ssh` 登录云服务器等操作。

2025 年 2 月 25 日，[Claude Code](https://www.anthropic.com/news/claude-3-7-sonnet) 低调发布。经过两年的飞速发展，大家似乎已经习惯了 AI 可以做出任何令人震惊的飞跃。在编程领域，从最初的代码补全，进化到了 **agentic coding**，一开始只是用 AI 生成 demo App，然后在社交媒体上炫耀自己用一句话生成的贪吃蛇游戏，严肃的程序员对 AI 编程持有批判怀疑态度。

2026 年初，形势悄然发生转变。

![programer](https://cdn.mahaoliang.tech/2026/20260113223628715.jpg)

[Ruby on Rails](https://rubyonrails.org/)的创造者，知名程序员 [DHH(David Heinemeier Hansson)](https://world.hey.com/dhh) 在 2025 年中的时候还认为“AI can’t code”，现在改变了看法，在最新的博客[《Promoting AI agents》](https://world.hey.com/dhh/promoting-ai-agents-3ee04945)中，DHH 认为在 2025 年 **AI agent** 产生跨越式发展，使用终端工具（Claude Code) 和 AI 协作式开发，已经能写生产级代码。

Redis 的作者，多才多艺的顶尖程序员 [Salvatore Sanfilippo](https://antirez.com/) 也在最新的博客[《Don't fall into the anti-AI hype》](https://antirez.com/news/158)中表达，AI 将**永远彻底改变编程**，而且比他预期的快得多。对大多数项目而言，**亲自写代码已不再是明智选择（除非只为乐趣）**。antirez 亲自实践的证据，几小时内完成了原本需要数周的 4 个任务：修改 `linenoise` 库支持 `UTF-8` 并构建测试框架；修复 `Redis` 测试中时序竞争/死锁等偶发故障；5 分钟生成 700 行纯 C `BERT` 嵌入推理库（性能仅比 `PyTorch` 低 15%）；20 分钟内让 AI 复现自己对 `Redis Streams` 的内部修改。

最令人震惊的，是 Linux 之父，传奇程序员 [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) 在上周末发布了自己的 Vibe Coding 项目[AudioNoise](https://github.com/torvalds/AudioNoise) ，使用 Google 的 [Antigravity](https://antigravity.google/) 实现音频样本的可视化功能。在提交信息中，Linus Torvalds 表示：“**Is this much better than I could do by hand? Sure is.**”

![AudioNoise](https://cdn.mahaoliang.tech/2026/20260113223853210.jpg)

程序大神们都已经拥抱 AI coding，我们还有什么理由拒绝呢。

但这和终端有什么关系？因为目前最好用的 **agentic coding** 工具仍然是[Claude Code](https://claude.com/product/claude-code)，或者有人认为是其开源平替版 [OpenCode](https://opencode.ai/)，它们都是命令行终端工作方式，程序员从每天面对 IDE，变成每天大多数时间都在终端中工作。原先的软件选择和设置，已经不能满足当前高强度使用的需求。

例如 macOS 原生 `terminal`，仅支持 `ANSI` 16 色与扩展 256 色，不支持 24 位真彩（**True Color**），渲染方式为 CPU 绘制，无 GPU 加速，大量输出或快速滚动时易出现卡顿。而现代终端，真彩和 GPU 加速都是基本功能。既然每天都要面对终端工作，当然不能凑合，必须找一个颜值高，使用流畅的现代终端。

原生 `terminal` 运行真彩测试的效果：

![terminal](https://cdn.mahaoliang.tech/2026/20260113224335237.jpg)

[Ghostty](https://ghostty.org/) 中运行真彩测试的效果：

![ghostty](https://cdn.mahaoliang.tech/2026/20260113224403817.jpg)

铺垫了这么多，终于要进入正题了，打造 2026 面向 **agentic coding** 的终端开发环境。

## 字体

首先准备基础环境，安装必要的字体。

### JetBrainsMono Nerd Font

[Nerd Fonts](https://www.nerdfonts.com) 是一个面向开发者的图标集合，包含了 10390+ 个来自主流图标集的图标。使用 `Nerd Fonts`，我们可以在单调的文本终端，显示丰富的图标，美观不输 GUI 界面，而且极具极客范儿。

![Nerd Fonts](https://cdn.mahaoliang.tech/2026/20260113224708727.svg)

[JetBrains Mono](https://www.jetbrains.com/lp/mono/) 非常流行，也是我最喜欢的编程字体，将 `JetBrains Mono` 和 `Nerd Fonts` 打包在一起，就得到新的字体 **JetBrainsMono Nerd Font**，兼顾编程可读性和图标可视化需求。[nerdfonts.com](https://www.nerdfonts.com/font-downloads) 提供了 `Nerd Fonts`和各种流行编程字体的打包字体，找到 [JetBrainsMono Nerd Font](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/JetBrainsMono.zip)，下载并安装。

### Sarasa Term SC

使用 Claude code 时必然要用到中文和 AI 交流，默认会使用系统自带的中文字体。**macOS** 上默认中文字体是 `PingFang SC`，虽然美观大方，但它不是等宽字体，和英文混排时，在终端上会显得有些凌乱。所以我们需要一款面向终端和代码编辑器的等宽中文字体。

[Sarasa Gothic](https://github.com/be5invis/Sarasa-Gothic) 是一款开源的多语言复合字体，基于 **Inter**、**Iosevka** 和 **思源黑体（Source Han Sans）** 三款经典字体整合而成，适配中日韩等多语言混排场景。`Sarasa Gothic`发布了一系列字体，其中 **Sarasa Term SC** 是面向终端（**Term**）的等宽（**Mono**）简体中文（**SC**）字体，强制保证 **中文宽度 = 2 × 英文宽度**，适合在终端显示中英文混排内容。

![Sarasa Term SC](https://cdn.mahaoliang.tech/2026/20260113225958444.jpg)

在 [Sarasa Gothic](https://github.com/be5invis/Sarasa-Gothic) 的 [relase 页面](https://github.com/be5invis/Sarasa-Gothic/releases/tag/v1.0.35)，**Single Family & Language TTF Package** 表格中，找到 **SC** 行，**Term** 列对应的字体，下载并安装。

## Ghostty

现代终端我选择了 [Ghostty](https://ghostty.org/)。除了真彩、GPU 加速等基本特性外，说一个 **agentic coding** 的刚需：给 Claude code 布置完任务后，它吭哧吭哧在那儿干活，你刷会儿微博娱乐。Claude code 完成当前任务后，需要等你下一步的指示，这时 **Ghostty** 会自动通知提醒，太有用了。

### 理解 AI agent

![AI agent](https://cdn.mahaoliang.tech/2026/20260113231147261.png)

说个题外话，如果你对 **AI agent** 没什么概念，用一下 Claude code 你马上就会明白。AI，或者说大语言模型，本质还是根据你的输入，预测下一个 `token`。最初的 AI 使用方式，就是对话，你输入提示词，AI 回答。具体到编程领域，你描述要求，AI 完成一段代码。但这样工作效率太低了，agent 出现了。

你可简单认为 agent 就是具有某种技能的员工，你想做一个 App，产品经理 agent 会和你讨论，明确需求和功能，输出产品功能描述文档，然后架构师根据产品文档，拆分任务，输出具体实现方案，程序员根据实现方案，开始编码干活，测试也是根据产品文档，对程序员的输出进行测试。除了第一步和产品经理讨论需要你参与，明确产品功能后，后面的角色，都会根据自己的职责，自动干活。相当于你雇佣了一堆人（agents），分工协作，听你指挥，帮你干活。

现在的 AI coding，早已不是一问一答，帮你完成个函数/测试，已经具备了一定的自主性。你像一个老板，去指挥这些 agents 干活。甚至现在已经有了手机上使用 Claude code 的方案，例如 [happy](https://github.com/slopus/happy)。想象一下，下班前你布置好任务，然后去健身，agents 开始干活，在完成了一个阶段的任务后，agents 需要请示老板，这时你手机收到通知，你看了一眼 AI 的汇报，然后下达下一步的指示。你也可以告诉它：今晚不要再打扰我，请根据计划，通宵工作，明早我检查成果。

**AI agent** 就是听你指挥，协作帮你完成任务的员工。

### Ghostty 配置

Ghostty 信奉 [Zero Configuration](https://ghostty.org/docs/config#zero-configuration-philosophy) 理念，但我还是做了少量配置。

```conf
font-family = ""
font-family = "JetBrainsMono Nerd Font Mono"
font-family = "Sarasa Term SC"
font-size = 16
font-thicken = true

window-height = 40
window-width = 160
window-padding-x = 5

theme = "TokyoNight"

macos-titlebar-style = tabs
macos-option-as-alt = true
background-opacity = 0.95
background-blur = true

shell-integration-features = ssh-terminfo,ssh-env,sudo
```

主字体设置为  **JetBrainsMono Nerd**，Ghostty 会首先尝试使用 **JetBrainsMono Nerd** 来渲染字符，当 Ghostty 遇到一个 **JetBrainsMono Nerd** 里面没有的字符，比如汉字，它会去查看配置列表里的下一个字体，即 **Sarasa Term SC**。

## Shell

仍然选择 `zsh` + `Oh My Zsh`，安装配置不变，请参考我以前的博客[《打造一个漂亮的命令行终端》](https://mahaoliang.tech/p/%E6%89%93%E9%80%A0%E4%B8%80%E4%B8%AA%E6%BC%82%E4%BA%AE%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%BB%88%E7%AB%AF/)和 [《Zsh 的安装和配置》](https://mahaoliang.tech/p/zsh-%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE/)。

## 命令行提示

命令行提示原先一直使用 [Powerlevel10k](https://github.com/romkatv/powerlevel10k)，最近发现更好的替代者 [Starship](https://starship.rs)。

Powerlevel10k 已经不再更新，进入维护阶段，而 Starship 还非常活跃，同时 Starship 支持 Linux、MacOS、Windows 等多个平台，这样我可以在各个平台上获得一致的体验。

先禁用 Powerlevel10k，并设置 ZSH_THEME="bira"

* **安装**

```bash
curl -sS https://starship.rs/install.sh | sh
```

* **主题配置**

主题选择 [Gruvbox Rainbow](https://starship.rs/presets/gruvbox-rainbow)，使用下面的命令安装主题：

```bash
starship preset gruvbox-rainbow -o ~/.config/starship.toml
```
同时，我希望对主题做一些自定义配置。打开 `~/.config/starship.toml` 文件，找到 `[directory]` 配置，修改设置如下：

```conf
truncation_length = 8
truncation_symbol = "…/"
truncate_to_repo = false
```
目录超过 8 层再截断。

默认情况下，如果是 Git 仓库，路径显示会自动截断使用 Git 仓库的根目录，我想看到完整的路径，所以设置 `truncate_to_repo` 为 `false`。

* **激活主题**

在 `~/.zshrc` 文件最后添加以下内容：

```zsh
# starship
if [[ "$TERM_PROGRAM" != "Apple_Terminal" ]]; then
    eval "$(starship init zsh)"
fi
```

由于 StarShip 需要真彩支持，所以只有使用非 macOS 原生 terminal 才加载 StarShip。最终的效果如下：

![Starship](https://cdn.mahaoliang.tech/2026/20260113234420087.png)

可以参考我的 `Oh My Zsh` [完整配置](https://gist.github.com/mahaoliang2005/30184866be88559bbc2c0e574e489662)。

## VS Code 配置

由于命令行提示符使用了 [Starship](https://starship.rs)，为了避免 `VS Code` 的终端出现乱码，需要设置`VS Code`的终端字体。打开配置文件 `Settings.json`中，增加或修改为如下配置：

```json
"terminal.integrated.fontFamily": "JetBrainsMono Nerd Font Mono",
```

另外，推荐两个 `VS Code` 主题：

* [Catppuccin](https://github.com/catppuccin/vscode)，编码时选择 `Catppuccin Mocha`，显示清晰，同时也不刺眼。

* [Flexoki](https://github.com/kepano/flexoki)，[Obsidian](https://obsidian.md/) 创始人 [@kepano](https://x.com/kepano) 创建的配色方案，宣纸的淡黄色，特别适合文字编辑。

![Flexoki](https://cdn.mahaoliang.tech/2026/20260114000635610.png)

## Jetbrains IDE 主题

[Catppuccin](https://github.com/catppuccin/catppuccin) 主题支持几乎所有的终端和代码编辑器，当然少不了 Jetbrains 的 IDE，同样可以选择 [Catppuccin](https://github.com/catppuccin/jetbrains) 主题。

## tmux 配置

原先隐约知道 [tmux](https://github.com/tmux/tmux) 这个工具，是为了长时间运行命令，即使断开 SSH 连接，命令也不会退出，下次连上去还可以恢复。

到了 **agentic coding** 时代，我突然明白了 tmux 的价值。想象这个场景：在实验室 ssh 连接到 Linux 服务器上使用 Claude Code 干活，离开时给它布置一个长任务，然后从 tmux session detach。吃饭完回到宿舍，使用自己的 MacBook 再次 ssh 到 Linux 服务器，tmux session attach，恢复 Claude Code 现场，看看它干的怎么样了，是否需要下一步指导。无法切换地点，不中断开发，太有用了！

对于 tmux，我同样选择了 [Flexoki](https://github.com/kepano/flexoki) 主题，并增加了如下配置：

```conf
# 设置 tmux 会话内部默认使用的终端类型为 tmux-256color
set -g default-terminal "tmux-256color"

# 强制 tmux 启用「真彩色（True Color）」支持
set -ag terminal-overrides ",*:RGB"

# 启用 tmux 的鼠标支持
set -g mouse
```

打开 tmux 的配置文件 `~/.tmux.conf`，先复制 [Flexoki](https://github.com/kepano/flexoki/blob/main/tmux/flexoki-light.tmuxtheme) 主题的内容，然后在末尾增加上面的配置。可以参考我的[完整配置](https://gist.github.com/mahaoliang2005/716cc6b052305b9ecc9e680cc258056d)。

全文完，希望对你有帮助，让我们在现代终端环境，愉快的 agentic coding！