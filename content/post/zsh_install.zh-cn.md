---
title: "Zsh 的安装和配置"
date: 2025-01-05T21:06:07+08:00
draft: false
tags: [macos,linux]
categories: [tech]
---
[Zsh](https://www.zsh.org/) 是一种专门为交互式使用而设计的 Shell，同时也是一种强大的脚本语言，集成了 bash、ksh 和 tcsh 的许多有用特性，并添加了许多独特的功能。

本文将指导您在 macOS 和 Linux 系统上安装 Zsh、Oh My Zsh 以及其常用插件，并展示如何配置 Oh My Zsh，以打造一个高效的命令行工作环境。

## 安装 Zsh

* macOS `brew install zsh`
* Ubuntu `sudo apt install zsh`
* RHEL `sudo yum update && sudo yum -y install zsh`

## 验证安装的 Zsh 版本

```bash
$ zsh --version
zsh 5.8.1 (x86_64-ubuntu-linux-gnu)
```

## 设置 Zsh 为缺省 shell

```bash
$ chsh -s $(which zsh)
```

退出并重新登录。

## 安装 Oh My Zsh
[Oh My Zsh](https://ohmyz.sh/) 是一个开源、社区驱动的 Zsh 配置管理框架，，提供了 300 多个可选插件和 140 多个主题，并且内置了自动更新工具。

使用下面的命令安装：

```bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 安装 zsh-autosuggestions

[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 为 zsh shell 提供了类似 Fish shell 的自动建议功能的插件，该插件可以根据历史记录和自动补全来为用户提供命令建议。

将插件 clone 到 `$ZSH_CUSTOM/plugins`：

```bash
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

然后在 `${HOME}/.zshrc` 启用插件：

```bash
plugins=(git zsh-autosuggestions)
```

在命令行输入命令时，zsh-autosuggestions 会根据命令历史或命令补全进行建议提示。那么如何接受建议呢？

Bash 和 Zsh 这样的 Unix shell 提供了两种主要的编辑模式：Emacs 模式和 Vi 模式，也就是说可以使用 Emacs 或 Vi 的快捷键来编辑命令行。Emacs 模式是缺省模式。

在 zsh-autosuggestions 的[缺省配置文件](https://github.com/zsh-users/zsh-autosuggestions/blob/master/src/config.zsh)中，定义接受建议的快捷键：

```bash
...
# Widgets that accept the entire suggestion
(( ! ${+ZSH_AUTOSUGGEST_ACCEPT_WIDGETS} )) && {
	typeset -ga ZSH_AUTOSUGGEST_ACCEPT_WIDGETS
	ZSH_AUTOSUGGEST_ACCEPT_WIDGETS=(
		forward-char
		end-of-line
		vi-forward-char
		vi-end-of-line
		vi-add-eol
	)
}
...
```

如果命令行处于 Emacs 模式，那么：

* `ctrl-f` 或 `ctrl-e` 跳到行尾接受当前的建议
* `option-f` 向前前进一个单词并接受建议

同样，如果命令行处于 vi 模式，那么就使用对应的 vi 键盘绑定接受建议。

## 配置 Oh My Zsh

Oh My Zsh 有非常多的[内置插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins)，你也可以安装第三方插件，就像上面安装的 zsh-autosuggestions。

Oh My Zsh 也内置了多个 [主题](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 供你选择。

我们可以编辑 `${HOME}/.zshrc`，配置 Oh My Zsh 的插件、主题，以及其他一些定制化设置。

```bash
...

# 设置主题
ZSH_THEME="bira"

# 启用插件
plugins=(git z zsh-autosuggestions)

# 命令别名
alias mkdir='mkdir -v'
alias mv='mv -v'
alias cp='cp -v'
alias rm='rm -v'
alias ln='ln -v'

# 配置zsh-autosuggestions
export ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#ff00ff,bg=cyan,bold,underline"
export ZSH_AUTOSUGGEST_STRATEGY=(history completion)
```
