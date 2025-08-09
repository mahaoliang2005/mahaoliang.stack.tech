---
title: "Python 项目管理最佳实践：uv 使用指南"
date: 2025-07-25T19:54:51+08:00
draft: false
tags: [python]
categories: [tech]
---
过去一个月，密集学习了各种 Python 相关的工具。

在我看来，Python 的知识分为两个方面，一个是 Python 语言本身，要学习它的语法，掌握基本数据结构，了解常用的模块，总之就是会写 Python 代码，解决实际问题。

另一个方面，是 Python 工程方面的知识。所谓工程，就是在开发大型项目时，如何让开发过程更高效：项目复杂了，如何组织代码结构；如何进行依赖管理；如何多人协作；如何做项目间隔离，避免依赖冲突；如何管理多个版本的 Python；如何构建、发布自己写的模块等等。总之就是，除了写代码之外，如何做，让开发 Python 的过程更高效。

学习的过程中，写了一系列的技术文章。从最开始用 [pyenv](https://mahaoliang.tech/p/%E5%9C%A8-macos-%E4%B8%8A%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8-pyenv/) 管理多版本，[venv](https://mahaoliang.tech/p/python-%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86venv-%E5%AE%9E%E7%94%A8%E6%8C%87%E5%8D%97/) 和 [Conda](https://mahaoliang.tech/p/%E4%B8%80%E6%96%87%E5%BD%BB%E5%BA%95%E6%90%9E%E6%87%82-python-%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86%E7%A5%9E%E5%99%A8-conda/) 管理虚拟环境，到阶段性的总结了[虚拟环境工具选择建议](https://mahaoliang.tech/p/python-%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%E9%80%89%E6%8B%A9%E5%BB%BA%E8%AE%AE/)，之后深入技术实现原理，探讨了[虚拟环境底层实现原理](https://mahaoliang.tech/p/%E6%8F%AD%E7%A7%98-python-%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86%E7%9A%84%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0/) 和 [Python 与 pip 的关系](https://mahaoliang.tech/p/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3-python-%E4%B8%8E-pip/)，之后总结了 [Python 项目管理的发展历程](https://mahaoliang.tech/p/%E7%8E%B0%E4%BB%A3-python-%E9%A1%B9%E7%9B%AE%E7%AE%A1%E7%90%86%E6%8C%87%E5%8D%97/)。

到今天，是这个系统的最后一篇：因为我会用 uv 代替前面介绍的所有工具，用 uv 管理 Python 项目的全流程。我决定了，以后的 Python 项目，都使用 uv 管理。

## uv 简介

[uv](https://github.com/astral-sh/uv) 是一个用 Rust 编写的 Python 打包和项目管理器，它将多个 Python 工具（如 pip、venv、pyenv 等）的功能整合到一个工具中，提供了一个统一、高效的 Python 开发流程。

实际上，uv 的实现并不是完全从头开始，而是**对 `venv` 和 `pip` 的高级封装**，对使用者提供更简单、更统一的接口。

本文将从零开始，使用 uv 完整地走完一个项目的开发、构建和安装流程。

## 安装 uv

### 使用官方脚本安装

推荐参考[uv 项目文档](https://docs.astral.sh/uv/getting-started/installation/)，使用官方脚本安装：

```bash
# macOS and Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

脚本执行成功后，新开一个 shell，执行 `uv --version` 命令，查看是否安装成功。

```bash
❯ uv --version
uv 0.8.3 (7e78f54e7 2025-07-24)
```

### PATH 环境变量

uv 被安装到用户目录的 `.local/bin` 目录下。

安装脚本会自动在 `.zshrc` 或 `.bashrc` 增加配置，将 `$HOME/.local/bin` 添加到 `$PATH` 环境变量中。

```bash
$ cat $HOME/.zshrc
...
. "$HOME/.local/bin/env"
```

查看`.local/bin/env` ，文件内容如下：

```bash
#!/bin/sh
# add binaries to PATH if they aren't added yet
# affix colons on either side of $PATH to simplify matching
case ":${PATH}:" in
    *:"$HOME/.local/bin":*)
        ;;
    *)
        # Prepending path in case a system-installed binary needs to be overridden
        export PATH="$HOME/.local/bin:$PATH"
        ;;
esac
```
可以看出，`.zshrc` 中这行命令 `. "$HOME/.local/bin/env"` 的作用，是将 `$HOME/.local/bin` 添加到 `$PATH` 中。

### Shell 自动补全

建议为 uv 命令启用 shell [自动补全功能](https://docs.astral.sh/uv/getting-started/installation/#shell-autocompletion)：

```bash
# zsh
echo 'eval "$(uvx --generate-shell-completion zsh)"' >> ~/.zshrc

# bash
echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
```

### 升级 uv

通过官方脚本安装 uv 后，可按需自行更新：

```bash
uv self update
```

## Python 版本管理

可以使用 `uv python list` 命令查看 uv 支持的所有 Python 版本：

```bash
❯ uv python list
cpython-3.14.0rc1-macos-aarch64-none                 <download available>
cpython-3.13.5-macos-aarch64-none                    <download available>
...
cpython-3.9.6-macos-aarch64-none                     /usr/bin/python3
...
pypy-3.8.16-macos-aarch64-none                       <download available>
...
graalpy-3.8.5-macos-aarch64-none                     <download available>
```
uv 会列出所用可用版本，包括系统自带的 Python。

我们可以指定安装某个版本 Python 版本：

```bash
❯ uv python install cpython-3.13.5
```

命令执行成功后，Python 将被安装到 `~/.local/bin` 目录下：

```bash
❯ ll ~/.local/bin
total 73784
-rw-r--r--  1 haoliangma  staff   328B  7 25 21:01 env
-rw-r--r--  1 haoliangma  staff   165B  7 25 21:01 env.fish
lrwxr-xr-x  1 haoliangma  staff    89B  7 25 21:07 python3.13 -> /Users/haoliangma/.local/share/uv/python/cpython-3.13.5-macos-aarch64-none/bin/python3.13
-rwxr-xr-x  1 haoliangma  staff    36M  7 25 05:09 uv
-rwxr-xr-x  1 haoliangma  staff   328K  7 25 05:09 uvx
```

可以看到，在 `~/.local/bin` 目录下建立了符号链接（Symbolic Link），指向 Python 的实际安装位置 `~/.local/share/uv/python`。

值得注意的是，符号链接名称带了具体的版本号 `python3.13`，也就是说，当前的 `python3` 命令，仍然是系统自带的 Python 环境。

```bash
❯ which python3
/usr/bin/python3
```

因为我们在项目中都会使用虚拟环境，所以并不在乎全局环境的 Python。这里我们使用 uv 安装的 Python，并不设置为全局环境，只供以后项目的虚拟环境使用。

后面还会看到，我们可以在 `~/.local/bin` 目录下安装可以独立运行的 Python 工具，而这些工具也自带了完整环境，并不会用到全局环境的 Python，工具之间的环境也是完全隔离的。

## 项目开发

### 创建项目

使用 `uv init` 命令可以快速初始化一个项目。我们可以指定项目名称，`init` 命令会自动创建项目目录，并生成项目结构。

```bash
❯ uv init -p 3.13 skycmd
Initialized project `skycmd` at `/Users/haoliangma/Documents/works/skycmd`
```

实际上，我们不必在上一步安装 Python，因为在项目初始化的时候，如果 `-p` 参数指定的 Python 版本不存在，`uv` 会自动下载并安装。

我们也可以在一个空的项目目录中执行 `uv init` 命令，这时会使用目录名作为项目名。

```bash
❯ mkdir skycmd
skycmd
❯ cd skycmd
❯ uv init -p 3.11
Initialized project `skycmd`
```

在初始化的项目目录下，`.python-version` 文件会记录项目的 Python 版本号。 

`pyproject.toml` 是最核心的项目配置文件，项目元数据、依赖项、构建配置等等都定义在这里。

### 运行初始化项目

`uv init` 为我们生成一个示例程序 `main.py`，使用 `uv run` 命令运行：

```bash
❯ uv run main.py
Using CPython 3.13.5
Creating virtual environment at: .venv
Hello from skycmd!
```

第一次运行项目，会自动创建虚拟环境 `.venv`。

### 添加依赖

我准备做一个查看天气的命令行工具，会用到 `requests` 处理 HTTP 请求，并使用 `click` 解析命令行参数。

使用 `uv add` 命令添加依赖：

```bash
uv add requests
uv add click
```

注意，不用执行 `source .venv/bin/activate` 显示的激活虚拟环境，`uv` 会自动将依赖安装到虚拟环境中。

安装的依赖，会自动记录在 `pyproject.toml` 文件中。

```toml
dependencies = [
    "click>=8.2.1",
    "requests>=2.32.4",
]
```

使用 `uv + pyproject.toml` 的方式管理依赖，和使用 `pip + requirements.txt` 的方式有本质区别。

`pyproject.toml` 只会记录**直接依赖**，而 `requirements.txt` 会记录**所有依赖**，包括直接依赖和间接依赖。

这两种方式的区别，在移除依赖项时就体现出来了：

- 使用 `uv remove requests`，会移除 `requests` 和所有 `requests` 的依赖项。
- 而 `pip uninstall requests` 只会移除 `requests`，留下了 `requests` 的依赖项。

### 配置构建工具

uv 支持多种构建打包工具，这里我们使用常见的 [setuptools](https://github.com/pypa/setuptools)。

参考 `setuptools` 的[官方文档](https://setuptools.pypa.io/en/latest/userguide/quickstart.html)，在 `pyproject.toml` 中添加 `build-system` 配置：

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

Python 不是脚本吗，为什么还要构建打包呢？

首先，因为我们的项目不可能总是一个简单的脚本，随着项目的复杂，可能会分为多个模块，每个模块提供独立、完整的功能。同时，项目还可以作为库发布出去，就像 `requests` 一样，让别人使用。这时，我们不可能将一个个的 Python 文件发给别人，而是需要使用一个构建打包工具，将我们的项目打包，方便别人安装使用。

另外，就算不发给别人使用，如果配置了 `[build-system]`，在开发阶段，我们不必使用 `pip install -e .` 在虚拟环境安装项目自身模块，而是直接使用 `uv run <project_name>` 命令就可以运行整个项目。

### 源码目录结构

`setuptools` 在构建时，会按照约定的源码目录结构，自动发现 Python 文件进构打包构建。

`setuptools` 默认支持两种源码目录布局：[src-layout](https://setuptools.pypa.io/en/latest/userguide/package_discovery.html#src-layout) 和 [flat-layout](https://setuptools.pypa.io/en/latest/userguide/package_discovery.html#flat-layout)。也就是说，按照这两种方式放置 Python 文件，不必做额外的配置，`setuptools` 都能自动找到。

我们使用简单的 `flat-layout`，调整项目目录结构如下：

```
skycmd
├── pyproject.toml  
├── ...
└── skycmd/
    ├── __init__.py
    ├── main.py
    └── weather/
         └── __init__.py
```

这里有几点说明请注意：

1. 首先，在项目的根目录下，创建与项目同名的目录 `skycmd`，用来存放 Python 源码。注意，**项目的根目录名**，根目录下存放**源码的目录名**，都要跟 `pyproject.toml` 定义的**项目名称**保持一致。

2. 将 uv 生成的 `main.py`，从项目根目录，移到源码目录 `skycmd` 下，作为项目执行入口。

3. 在 `skycmd` 目录下，可以创建子目录，作为子模块。例如可以创建 `weather` 目录，用于放置获取天气信息的模块代码。

4. 源码目录及其子目录，每层目录下都要建一个空的 `__init__.py` 文件。因为目录下如果有 `__init__.py` 文件，Python 会认为它是一个模块。有了 `__init__.py` 文件，每层目录都会被自动识别为模块。

### 配置项目执行入口

在 `pyproject.toml` 文件中，添加 `[project.scripts]` 配置项，指定项目执行入口。

```toml
[project.scripts]
skycmd = "skycmd.main:main"
```

`skycmd.main:main` 表示 `skycmd` 模块下的 `main.py` 文件中的 `main` 函数。

现在可以使用 `uv run skycmd` 命令来运行项目了：

```bash
❯ uv run skycmd
      Built skycmd @ file:///Users/haoliangma/Documents/works/skycmd
Installed 1 package in 7ms
Hello from skycmd!
```

因为前面我们前面配置了构建工具，现在又配置了执行入口，在运行 `uv run skycmd` 时，会自动进入虚拟环境，构建，将构建的包安装到虚拟环境，然后运行项目。

注意，整个过程，我们都只使用了 `uv` 命令，没有显示的执行 `source .venv/bin/activate` 命令来激活虚拟环境，`uv` 命令会自动激活虚拟环境。

同时，我们指定的是执行入口 `skycmd`，而不再是 `main.py`。

现在看好像差别不大，但如果项目包含了多个模块，`uv run skycmd` 命令会自动将项目的所有模块安装到虚拟环境，然后运行入口函数：`skycmd` 模块的 `main.py` 文件中的 `main` 函数。如何运行的是 `main.py`，将不会自动安装项目自身编写的模块。

### 开发服务模块

搭建好了整个项目结构，我们可以开发提供获取天气信息的模块。

实现很简单，构建 HTTP 请求访问 [wttr.in](https://wttr.in) ，提供城市名作为参数，HTTP 响应内容就是城市天气信息。

在 `skycmd/weather` 目录下，创建 `service.py` 文件，并添加如下代码：

```python
import requests

def get_weather_from_wttr(city_name):
    try:
        url = f"https://wttr.in/{city_name}?m&format=3"
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.text.strip()
    except requests.exceptions.RequestException as e:
        return f"Error getting weather information: {e}"


def get_detailed_weather_from_wttr(city_name):
    try:
        url = f"https://wttr.in/{city_name}?m"
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.text
    except requests.exceptions.RequestException as e:
        return f"Error getting detailed weather information: {e}"
    
if __name__ == "__main__":
    weather_info = get_weather_from_wttr("Shenzhen")
    print(weather_info)
```

记得要在 `skycmd/weather` 目录下要创建一个空的 `__init__.py` 文件。当前的目录结构如下：

```
skycmd
├── pyproject.toml  
├── ...
└── skycmd/
    ├── __init__.py
    ├── main.py
    └── weather/
        ├── __init__.py
        └── service.py
```

我们可以在 `skycmd/weather/server.py` 中添加测试方法，使用 `uv run` 运行该模块，验证编写的服务。

```bash
❯ uv run skycmd/weather/service.py
Shenzhen: ⛅️  +34°C
```

### 开发入口程序

获取天气信息的模块已经完成，现在我们编写入口程序。

修改 `skeycmd/main.py`文件，内容如下：

```python
import requests
import os
import sys
import click
from skycmd.weather.service import get_weather_from_wttr, get_detailed_weather_from_wttr

@click.command()
@click.argument('city', type=str, required=False)
@click.option('-v', '--verbose', is_flag=True, help='显示详细天气信息')
def main(city, verbose):
    """
    命令行天气工具 - 获取指定城市的天气信息
    
    \b
    使用示例：
      skycmd          # 获取帮助信息
      skycmd Shenzhen  # 获取深圳的天气
      skycmd -v Shenzhen  # 获取深圳的详细天气信息
    """
    if not city:
        # 如果没有提供城市名，显示帮助信息
        ctx = click.get_current_context()
        click.echo(ctx.get_help())
        ctx.exit()
    
    if verbose:
        # 显示详细天气信息
        weather_info = get_detailed_weather_from_wttr(city)
        print(weather_info)
    else:
        # 显示简单天气信息
        weather_info = get_weather_from_wttr(city)
        print(weather_info)


if __name__ == "__main__":
    main()
```

现在可以使用 `uv run skycmd` 运行项目：

```bash
❯ uv run skycmd
Usage: skycmd [OPTIONS] [CITY]

  命令行天气工具 - 获取指定城市的天气信息

  使用示例：
    skycmd          # 获取帮助信息
    skycmd Shenzhen  # 获取深圳的天气
    skycmd -v Shenzhen  # 获取深圳的详细天气信息

Options:
  -v, --verbose  显示详细天气信息
  --help         Show this message and exit.

❯ uv run skycmd Shenzhen
Shenzhen: ⛅️  +34°C
```

因为前面已经在 `pyproject.toml` 中配置了执行入口 `skycmd = "skycmd.main:main"`，所以使用 `uv run` 的时候，指定的是 `skycmd`，而不是 Python 文件名 `main.py`。

现在我们简单的修改一下程序，增加显示简单天气时的输出内容：

```python
        # 显示简单天气信息
        weather_info = get_weather_from_wttr(city)
        print(f"🌍 天气信息：")
        print(weather_info)
```

修改完成后，再次使用 `uv run skycmd Shenzhen` 运行，可以查看到改动后的效果：

```bash
❯ uv run skycmd Shenzhen
🌍 天气信息：
Shenzhen: ⛅️  +34°C
```

完美！开发过程顺畅丝滑。

## 构建和安装工具

如果想使用我们开发的命令行工具 `skycmd`，必须每次都要到项目目录下运行 `uv run skycmd`，这样有点麻烦。

我们可以先使用 `uv build`，将整个项目打包成标准的 Python 包：

```bash
❯ uv build
Building source distribution...
...
Successfully built dist/skycmd-0.1.0.tar.gz
Successfully built dist/skycmd-0.1.0-py3-none-any.whl
```

可以看到，构建成功后，在 `dist` 目录下生成了两个文件：

* `skycmd-0.1.0.tar.gz`：源代码分发包
* `skycmd-0.1.0-py3-none-any.whl`：二进制分发包

接着可以使用 `uv tool install` 命令，将 `skycmd` 安装到用户目录：

```bash
❯ uv tool install dist/skycmd-0.1.0-py3-none-any.whl
Resolved 7 packages in 563ms
Prepared 1 package in 4ms
Installed 7 packages in 7ms
 + certifi==2025.7.14
 + charset-normalizer==3.4.2
 + click==8.2.1
 + idna==3.10
 + requests==2.32.4
 + skycmd==0.1.0 (from file:///Users/haoliangma/works/skycmd/dist/skycmd-0.1.0-py3-none-any.whl)
 + urllib3==2.5.0
Installed 1 executable: skycmd tool install
```

`uv tool install` 安装的是二进制分发包 `dist/skycmd-0.1.0-py3-none-any.whl`。

命令执行成功，将在 `~/.local/bin` 目录下生成 `skycmd` 命令。

```bash
❯ ll ~/.local/bin
...
lrwxr-xr-x  1 haoliangma  staff    57B  7 25 22:34 skycmd -> /Users/haoliangma/.local/share/uv/tools/skycmd/bin/skycmd
-rwxr-xr-x  1 haoliangma  staff    36M  7 25 05:09 uv
-rwxr-xr-x  1 haoliangma  staff   328K  7 25 05:09 uvx
```

这个 `skycmd` 是一个符号链接，指向 `~/.local/share/uv/tools/skycmd/bin/skycmd`。

所以，`skycmd` 工具的实际文件存放在 `~/.local/share/uv/tools/skycmd/`目录中。查看这个目录的内容：

```bash
❯ tree -L 2 ~/.local/share/uv/tools/skycmd/
/Users/haoliangma/.local/share/uv/tools/skycmd/
├── bin
│   ├── activate
│   ├── ...
│   ├── python -> /Users/haoliangma/.local/share/uv/python/cpython-3.13.5-macos-aarch64-none/bin/python3.13
│   └── skycmd
├── CACHEDIR.TAG
├── lib
│   └── python3.13
├── pyvenv.cfg
└── uv-receipt.toml
```

可以看出，这个目录包含了一个完整的 Python 虚拟环境。所以虽然 `skycmd` 安装到了用户目录，但它使用自己的虚拟环境，指定了 Pythonn 版本，依赖也安装在自己的环境中。这样，工具之间的环境是完全隔离的，不会相互影响。

现在，可以在命令行中使用 `skycmd` 命令了：

```bash
❯ skycmd Shenzhen
🌍 天气信息：
Shenzhen: ⛅️  +29°C
```

不想用了，记得删除：

```bash
❯ uv tool uninstall skycmd
Uninstalled 1 executable: skycmd
```

完美！

## 总结

本文详细记录了使用 `uv` 管理 Python 项目的全流程（虽然没有讲到模块发布）。

`uv` 会自动创建虚拟环境，执行 `uv` 命令会自动在虚拟环境中运行，不需要显示的激活。

所以使用 `uv` 时要记得：**一切操作都要使用 `uv`，最好不要 `uv` 和 `pip` 混用。**

### 整理本文提到的主要命令

| 功能 | 命令 | 描述 |
| --- | --- | --- |
| **更新**| `uv self update` | 将 `uv` 更新到最新版本。 |
| **Python 版本管理** | `uv python list` | 列出所有可用的 Python 版本，包括已安装和可下载的。 |
| | `uv python install <version>` | 下载并安装指定版本的 Python。 |
| **项目初始化** | `uv init -p <python_version> <project_name>` | 初始化一个新的 Python 项目，可指定 Python 版本并自动创建项目目录和基础文件，如 `pyproject.toml`。 |
| **运行代码** | `uv run <file_or_script>` | 在项目的虚拟环境中运行 Python 脚本或已配置的入口。首次运行时会自动创建虚拟环境。 |
| **依赖管理** | `uv add <package>` | 向项目中添加依赖，并自动更新 `pyproject.toml` 文件。 |
| | `uv remove <package>` | 从项目中移除依赖及其相关子依赖。 |
| **项目构建** | `uv build` | 将项目打包成标准的源代码分发包 (`.tar.gz`) 和二进制分发包 (`.whl`)。 |
| **工具安装** | `uv tool install <package_or_wheel>` | 将一个 Python 包（如本地构建的 `.whl` 文件）作为一个独立的命令行工具安装。`uv` 会为其创建一个隔离的虚拟环境。 |
| | `uv tool uninstall <tool_name>` | 卸载已安装的命令行工具。 |

### 项目开发流程

本文通过创建一个名为 `skycmd` 的命令行天气查询工具，演示了使用 `uv` 的完整开发流程：

1.  **初始化项目**: 使用 `uv init` 创建项目结构和 `pyproject.toml` 配置文件。
2.  **添加依赖**: 使用 `uv add requests click` 添加项目所需的第三方库。
3.  **配置构建系统**: 在 `pyproject.toml` 中配置 `setuptools` 作为构建工具，以便后续的构建和打包。
4.  **组织源码**: 采用 `flat-layout` 目录结构，将源代码放置在与项目同名的子目录中。
5.  **配置执行入口**: 在 `pyproject.toml` 的 `[project.scripts]` 部分指定项目的命令行入口，使得可以使用 `uv run skycmd` 来运行。
6.  **开发与调试**: 在开发过程中，直接使用 `uv run` 来测试和运行代码，`uv` 会自动处理环境和依赖。
7.  **构建与安装**: 开发完成后，使用 `uv build` 将项目打包，然后使用 `uv tool install` 将其安装为系统级的命令行工具，方便在任何地方调用。

本文的示例代码在 [GitHub](https://github.com/mahaoliang2005/skycmd)，希望本文对你有帮助。