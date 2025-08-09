---
title: "在 macOS 上安装和使用 pyenv"
date: 2025-06-14T13:18:30+08:00
draft: false
tags: [python]
categories: [tech]
---
作为 Python 初学者，使用 [pyenv](https://github.com/pyenv/pyenv) 来管理多版本 Python 环境是一个明智的选择。pyenv 允许你轻松安装、切换和管理多个 Python 版本，同时保护系统自带的 Python 不被修改。本指南将详细介绍在 macOS 上安装和使用 pyenv 的全过程，包括安装步骤、管理不同 Python 版本、保护系统 Python 等操作方法。

## 准备工作

### 检查 Homebrew 安装

在开始安装 pyenv 之前，首先需要确保你已经安装了 Homebrew。Homebrew 是 macOS 上的包管理器，方便我们安装各种工具和依赖。可以在终端中运行以下命令检查：

```bsh
$ brew --version
Homebrew 4.5.8
```

### 安装 Xcode 命令行工具

macOS 上编译安装 Python 需要一些开发工具，这些工具可以通过 Xcode 命令行工具提供。运行以下命令安装：

```bash
$ xcode-select --install
```

这会弹出一个安装窗口，按照提示完成安装即可。

## 安装 pyenv

### 使用 Homebrew 安装 pyenv

在 macOS 上安装 pyenv 最简单的方法就是使用 Homebrew。在终端中运行以下命令：

```bash
$ brew install pyenv
```
这一步会下载并安装 pyenv 及其依赖。安装完成后，你可以通过以下命令验证安装是否成功：

```bash
$ pyenv --version
pyenv 2.6.3
```

如果看到版本号，说明安装成功。

### 配置环境变量

安装完成后，需要将 pyenv 添加到你的 Shell 配置文件中，以便在任何终端会话中都能使用。根据你使用的 Shell 类型（zsh 或 bash），打开相应的配置文件。以 zsh 为例，打开 `${HOME}/.zshrc` 文件，在文件末尾添加以下内容：

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - zsh)"
```

这些配置将：

1. 设置 pyenv 的根目录
2. 将 pyenv 的二进制目录添加到系统 PATH 中
3. 初始化 pyenv 环境

保存文件后，在终端中运行以下命令使配置生效：

```bash
source ${HOME}/.zshrc
```

## 管理 Python 版本

### 查看可用 Python 版本

安装 pyenv 后，你可以查看所有可用的 Python 版本：

```bash
$ pyenv install --list
Available versions:
  2.1.3
  2.2.3
  2.3.7
  2.4.0
  2.4.1
...
```

这会列出所有可通过 pyenv 安装的 Python 版本，包括最新版本和旧版本。

### 安装特定版本的 Python

要安装特定版本的 Python，使用以下命令：

```bash
$ pyenv install <version>
```

例如，要安装 Python 3.13.5，可以运行：

```bash
$ pyenv install 3.13.5
```

安装过程可能需要一些时间，因为 pyenv 会从源代码编译 Python。你会看到类似以下的提示：

```bash
python-build: use openssl@3 from homebrew
python-build: use readline from homebrew
Downloading Python-3.13.5.tar.xz...
-> https://www.python.org/ftp/python/3.13.5/Python-3.13.5.tar.xz
Installing Python-3.13.5...
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.13.5 to /Users/haoliangma/.pyenv/versions/3.13.5
```

### 查看已安装的 Python 版本

安装完成后，可以使用 `pyenv versions` 查看已安装的 Python 版本：

```bash
$ pyenv versions
  system
  2.7.18
* 3.11.8 (set by /Users/haoliangma/.pyenv/version)
  3.13.5
```

第一行 `system` 表示系统自带的 Python 版本，通常由操作系统预装。当前未被激活（没有 * 标记）

第二行 `2.7.18` 表示通过 pyenv 安装的 Python 2.7.18 版本。当前未被激活（没有 * 标记）。

第三行 `* 3.11.8 (set by /Users/haoliangma/.pyenv/version)` 表示当前激活的 Python 版本，该版本是通过全局配置文件 `/Users/haoliangma/.pyenv/version` 设置的默认版本。

第四行 `3.13.5` 表示通过 pyenv 安装的 Python 3.13.5 版本，同样没有被激活。

### 设置全局 Python 版本

要设置系统默认的 Python 版本，可以使用以下命令：

```bash
$ pyenv global <version>
```

例如，要将 Python 3.13.5 设置为全局默认版本：

```bash
$ pyenv global 3.13.5
```

再次使用 `pyenv versions` 查看已安装的版本：

```bash
$ pyenv versions
  system
  2.7.18
  3.11.8
* 3.13.5 (set by /Users/haoliangma/.pyenv/version)
```

发现 3.13.5 版本已经被激活。

可以通过以下命令验证：

```bash
$ python --version
Python 3.13.5
```

### 设置局部 Python 版本

在项目目录中，你可以设置特定于该项目的 Python 版本。进入项目目录，运行：

```bash
$ pyenv local <version>
```

这会在当前目录下创建一个名为`.python-version`的文件，记录当前项目使用的 Python 版本。当你进入该目录时，pyenv 会自动切换到指定的 Python 版本。例如：

```bash
$ cd pythonprojects
$ pyenv local 3.11.8
$ pyenv versions
  system
  2.7.18
* 3.11.8 (set by /Users/haoliangma/works/pythonprojects/.python-version)
  3.13.5

$ cd ..
$ pyenv versions
  system
  2.7.18
  3.11.8
* 3.13.5 (set by /Users/haoliangma/.pyenv/version)
```

### 临时使用特定版本

如果你只需要在当前终端会话中临时使用某个 Python 版本，可以使用：

```bash
$ pyenv shell <version>
```

例如：

```bash
$ pyenv versions
  system
  2.7.18
  3.11.8
* 3.13.5 (set by /Users/haoliangma/.pyenv/version)
$ pyenv shell 3.11.8
$ pyenv versions
  system
  2.7.18
* 3.11.8 (set by PYENV_VERSION environment variable)
  3.13.5
```

这会覆盖全局和局部设置，仅在当前终端会话中生效。要恢复到之前的设置，可以使用：

```bash
$ pyenv shell --unset
```

### 卸载 Python 版本

当你不再需要某个 Python 版本时，可以使用以下命令卸载：

```bash
$ pyenv uninstall <version>
```

## 保护系统 Python

macOS 系统自带了一个 Python 解释器，通常位于`/usr/bin/python3`。这个 Python 版本是系统正常运行所必需的，修改或删除它可能导致系统不稳定或某些功能无法正常工作。因此，保护系统 Python 非常重要。

pyenv 不会自动管理或修改系统 Python。当你安装新的 Python 版本时，它们会被安装在`~/.pyenv/versions`目录下，而不是系统路径中。这意味着系统 Python 始终保持不变，不会受到 pyenv 安装的版本的影响。

你可以通过以下命令验证系统 Python 是否未被修改：

```bash
$ which python
/Users/haoliangma/.pyenv/shims/python
```

可以看出，你当前使用的是 pyenv 管理的 Python 版本。

为了确保系统 Python 不被覆盖，你应该避免使用`sudo`安装或升级 Python。此外，在设置全局 Python 版本时，应确保不将系统 Python 设置为全局版本。

如果你不小心覆盖了系统 Python 的某些行为，你可以通过重新安装 Xcode 命令行工具来恢复。运行：

```bash
$ xcode-select --install
```

这会重新安装系统工具，包括系统 Python。

## 总结

通过本指南，你应该已经掌握了在 macOS 上使用 pyenv 管理 Python 版本的基本技能。以下是关键点回顾：

1.  **安装 pyenv**：使用 Homebrew 安装 pyenv 和相关插件，确保正确配置环境变量。

2.  **管理 Python 版本** 

| 命令                      | 描述                |
| ------------------------- | ------------------- |
| pyenv install -list      | 查看可安装版本      |
| pyenv install <versio>    | 安装指定版本        |
| pyenv versions            | 查看已安装版本      |
| pyenv global  <versio>    | 设置全局默认版本    |
| pyenv local <versio>      | 为当前目录设置版本  |
| pyenv shell  <version>    | 为当前 Shell 设置版本 |
| pyenv uninstall <version> | 卸载版本            |


3.  **保护系统 Python**：pyenv 不会自动管理系统 Python，确保系统 Python 不被覆盖是使用 pyenv 的重要原则。


通过使用 pyenv，你可以在保持系统 Python 完整的同时，灵活地管理多个 Python 版本，使你的 Python 开发更加高效和安全。