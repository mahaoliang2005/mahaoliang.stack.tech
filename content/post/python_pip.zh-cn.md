---
title: "深入理解 Python 与 pip"
date: 2025-07-20T08:05:26+08:00
draft: false
tags: [python]
categories: [tech]
---

## Python 与 pip 的关系

刚开始学 python 的时候，知道 python 是一个解释器，而 pip 是一个包管理器，以为它们的“地位”是平等的。然而随着了解的深入才发现，**pip 本身就是一个 Python 包，它的运行，完全依赖于 python 解释器。**

简单来说，它们的关系可以这样定义：

* **Python 是解释器**：它是一种高级编程语言，我们编写的 `.py` 脚本需要通过 Python 解释器来执行，将代码转换成机器可以理解的指令。没有 Python 解释器，我们写的 Python 代码就是一堆没有生命的文本文件。

* **pip 是包管理器**：可以把它想象成 Python 世界的“应用商店 (App Store)”。它是一个命令行工具，专门用来查找、下载、安装、卸载和管理 Python 的第三方软件包（也称为库或模块）。

最关键的一点是：**pip 本身也是一个用 Python 编写的包**。这意味着 `pip` 的运行离不开 `python` 解释器的支持，它们是一个密不可分的组合，Python 提供了运行环境，而 pip 则极大地丰富 Python 的能力。

### 为什么需要包管理？

在没有包管理的早期，开发者如果想使用别人写好的代码，可能需要手动去网站下载源码压缩包，解压后自己想办法放到项目特定的目录下，如果这个代码还依赖其他代码，整个过程将成为一场噩梦。`pip` 的出现彻底改变了这一切：

* **代码重用与效率提升**：当需要实现一个复杂的功能时，比如发送网络请求，很大概率已经有非常成熟的第三方库存在了。通过一条简单的 `pip install requests` 命令，你就能获得强大的网络请求能力，无需从零开始编写复杂的底层代码。

* **自动处理依赖关系**：一个库通常会依赖其他多个库才能正常工作。例如，你安装 `A` 库，`A` 库可能需要 `B` 库的 `1.5` 以上版本和 `C` 库的任意版本。如果手动管理，这将变得极其繁琐且容易出错。`pip` 则能自动分析这些依赖关系，将所有需要的“配料”一次性、按正确的版本要求准备妥当。

* **确保版本一致性**：`pip` 允许我们将项目所有依赖包记录在一个 `requirements.txt` 文件中。这样，任何人在任何地方，都可以通过这个文件创建出完全相同的运行环境。

### 捆绑安装

为了强调 `pip` 的重要性，自 Python 3.4 版本以来，官方的 Python 安装程序已经默认将 `pip` 捆绑在内。这意味着，当你从官方渠道下载并安装 Python 时，`pip` 工具也一并被安装到了你的系统中。

## 精准定位：我的 Python 和 pip 在哪里？

在日常开发中，我们常常会在电脑上安装多个版本的 Python，例如系统自带的 Python、通过 Homebrew 安装的 Python、以及使用 `pyenv` 等版本管理工具安装的多个 Python。这就带来了一个核心问题：当我们在终端里敲下 `python3` 或 `pip3` 命令时，我们到底在调用哪一个？

### 快速定位：`which` 和 `where` 命令

最直接的定位方法是使用操作系统提供的路径查找命令。

* 在 macOS 或 Linux 系统中，我们可以使用 `which` 命令：

```bash
# 查看 python3 可执行文件的路径
which python3

# 查看 pip3 可执行文件的路径
which pip3
```

* 在 Windows 系统中，对应的命令是 `where`：
    
```bash
# 查看 python3 可执行文件的路径
where python3

# 查看 pip3 可执行文件的路径
where pip3
```

这些命令会搜索系统的环境变量 `PATH`，并返回找到的第一个匹配的可执行文件的完整路径。

### 真正的 Python 解释器在哪里？

当你使用像 `pyenv` 这样的 Python 版本管理工具时，`which` 命令的结果可能会让你感到困惑：

```bash
❯ which python3
/Users/haoliangma/.pyenv/shims/python3
```

这个路径指向的并不是一个真实的 Python 解释器，而是一个叫做“shim”的转发脚本。当你执行 `python3` 命令时，实际上是这个“shim”接到了指令。它会检查你当前目录或全局设置，判断你希望使用哪个版本的 Python（例如 `3.10.18`），然后再将你的命令无缝地转接给那个版本对应的真实解释器。

那么，如何才能找到真正的 Python 解释器？

执行下面的命令，返回真实解释器路径：

```bash
❯ python3 -c "import sys; print(sys.executable)"
```

通过这个命令，我们可以清晰地看到真相：

```bash
# 由 pyenv 管理的 Python
❯ python3 -c "import sys; print(sys.executable)"
/Users/haoliangma/.pyenv/versions/3.10.18/bin/python3

# macOS 系统自带的 Python
❯ python3 -c "import sys; print(sys.executable)"
/Applications/Xcode.app/Contents/Developer/usr/bin/python3
```

这个路径才是我们正在使用的 Python 解释器所在的位置。

### 确认 pip 的真实位置

同样地，我们也需要确认 `pip` 的真实位置。可以使用 `pip3 --version` 命令：

```bash
# 由 pyenv 管理的 Python
❯ pip3 --version
pip 23.0.1 from /Users/haoliangma/.pyenv/versions/3.10.18/lib/python3.10/site-packages/pip (python 3.10)

# macOS 系统自带的 Python
❯ pip3 --version
pip 21.2.4 from /Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages/pip (python 3.9)
```

我们还可以用 `pip` 自己来“调查”自己，因为 `pip` 本身也是一个包：

```bash
# 由 pyenv 管理的 Python
❯ pip3 show pip
Name: pip
Version: 23.0.1
Summary: The PyPA recommended tool for installing Python packages.
Home-page: https://pip.pypa.io/
Author: The pip developers
Author-email: distutils-sig@python.org
License: MIT
Location: /Users/haoliangma/.pyenv/versions/3.10.18/lib/python3.10/site-packages
Requires: 
Required-by: 

# macOS 系统自带的 Python
❯ pip3 show pip
Name: pip
Version: 21.2.4
Summary: The PyPA recommended tool for installing Python packages.
Home-page: https://pip.pypa.io/
Author: The pip developers
Author-email: distutils-sig@python.org
License: MIT
Location: /Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages
Requires: 
Required-by: 
```

## `pip install` 的三个目的地

当我们执行 `pip install <package_name>` 时，这个包究竟被安装到哪里了？

`pip install` 有三个不同的目的地：**全局 Site-Packages**、**用户 Site-Packages** 和**虚拟环境 Site-Packages**。

### 全局安装：便捷但危险的默认选项

这是最直接的安装方式，但通常也是最不推荐的方式。

当你的全局 Python 环境拥有写入权限时（例如，使用 `pyenv`、Homebrew 或在 Windows 上默认安装的 Python），执行 `pip install package_name`，会将 package 安装到全局 `site-packages` 目录下。

如何知道全局 site-packages 具体在哪里？

可以通过 `pip show` 查看已安装包的位置，得知 site-packages 的位置：

```bash
❯ pip3 install pyfiglet
Successfully installed pyfiglet-1.0.3

❯ pip3 show pyfiglet
Name: pyfiglet
Version: 1.0.3
...
Location: /Users/haoliangma/.pyenv/versions/3.10.18/lib/python3.10/site-packages
...
```

输出中的 `Location` 字段地告诉我们，`pyfiglet` 被安装的具体位置。由于是在全局环境执行的安装，这个位置就是全局 `site-packages` 目录。

在安装之前，你可以通过编程方式，预先知道全局 `site-packages` 的路径：

```bash
❯ python3 -c "import site; print(site.getsitepackages())"
['/Users/haoliangma/.pyenv/versions/3.10.18/lib/python3.10/site-packages']
```
对于 macOS 自带的 Python，`site.getsitepackages()` 可能会返回一个包含多个路径的列表：

```bash
python3 -c "import site; print(site.getsitepackages())"
['/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages', '/Applications/Xcode.app/Contents/Developer/AppleInternal/Library/Python/3.9/site-packages', '/Library/Python/3.9/site-packages', '/AppleInternal/Library/Python/3.9/site-packages', '/AppleInternal/Tests/Python/3.9/site-packages']
```

 这意味着 Python 在查找全局包时，会按照列表中的顺序依次搜索这些目录。这是一种分层系统，允许系统本身、Xcode 工具链以及管理员安装的包共存。

我们应该**极力避免**全局安装。这种方式会“污染”你的主 Python 环境，一旦项目增多，极易导致不同项目间的依赖版本冲突。

macOS 系统自带的 Python，普通用户没有权限进行全局安装，除非你明确使用 `sudo` 执行安装。

### 用户目录安装

在两种情况下，`pip install` 会将包安装到用户级别的 `site-packages` 目录下。

**第一种情况是被动触发**，当执行全局安装没有权限时，pip 会自动将包安装到用户级别的 `site-packages`，例如使用 macOS 系统自带的 Python 时：

```bash
❯ pip3 install pyfiglet
Defaulting to user installation because normal site-packages is not writeable
...
Successfully installed pyfiglet-1.0.3
```

全局目录不可写入，但安装还是成功了，实际上是将包安装到了用户目录。查看刚刚的安装包，可以知道用户目录的具体位置。

```bash
❯ pip3 show pyfiglet
Name: pyfiglet
Version: 1.0.3
Summary: Pure-python FIGlet implementation
Home-page: https://github.com/pwaller/pyfiglet
Author: Peter Waller (Thanks to Christopher Jones and Stefano Rivera)
Author-email: p@pwaller.net
License: MIT
Location: /Users/haoliangma/Library/Python/3.9/lib/python/site-packages
Requires: 
Required-by: 
```

**第二种情况是**，通过 `--user` 参数，明确要求将包安装在自己的用户目录下。

```bash
❯ pip3 install --user pyfiglet
❯ pip3 show pyfiglet
Name: pyfiglet
Version: 1.0.3
Summary: Pure-python FIGlet implementation
Home-page: https://github.com/pwaller/pyfiglet
Author: Peter Waller (Thanks to Christopher Jones and Stefano Rivera)
Author-email: p@pwaller.net
License: MIT
Location: /Users/haoliangma/.local/lib/python3.10/site-packages
Requires: 
Required-by:
```

这次使用的是 pyenv 管理的 python，包被安装到用户的 `.local/lib/python3.10/site-packages` 目录中。

我们还发现，不同方式安装的 Python，可能有不同的用户目录。不用安装包，可以执行`python3 -m site --user-site` 命令，获得当前 Python 的用户目录：

```bash
# pyenv 管理的 python
❯ python3 -m site --user-site
/Users/haoliangma/.local/lib/python3.10/site-packages

# macOS 系统自带的 python
❯ python3 -m site --user-site
/Users/haoliangma/Library/Python/3.9/lib/python/site-packages
```

### 虚拟环境安装

虚拟环境是为每个项目创建一个独立的，与其他项目隔离的“沙盒”。

在**已激活 (activated)** 的虚拟环境中执行 `pip install`，会将包安装到虚拟环境的 site-packages 目录下。

```bash
# 在项目目录下创建一个名为 .venv 的虚拟环境
❯ python3 -m venv .venv

# 激活它 (macOS/Linux)
❯ source .venv/bin/activate

# 安装包
(.venv) ❯ pip3 install pyfiglet

# 查看包的安装位置
(.venv) ❯ pip3 show pyfiglet
Name: pyfiglet
Version: 1.0.3
...
Location: /Users/haoliangma/works/demo/.venv/lib/python3.10/site-packages
...
```
`Location` 清晰地指向了我们刚刚创建的 `.venv` 目录内部，与全局或用户目录的路径毫无关系。

在激活虚拟环境的状态下，我们再次运行之前的查询命令，查看当前的 site-packages 路径：

```bash
(.venv) ❯ python3 -c "import site; print(site.getsitepackages())"
['/Users/haoliangma/works/demo/.venv/lib/python3.10/site-packages']
```

## 查看与管理已安装的包

### `pip list`

`pip list` 会列出当前 Python 环境中所有已安装的包及其版本号。

使用 pyenv 管理的 python，在全局环境中运行`pip list`，会列出全局和用户目录下的包。

```bash
❯ pip3 list
Package    Version
---------- -------
pip        23.0.1
pyfiglet   1.0.3
setuptools 65.5.0

❯ pip3 show pyfiglet
Name: pyfiglet
Version: 1.0.3
Summary: Pure-python FIGlet implementation
Home-page: https://github.com/pwaller/pyfiglet
Author: Peter Waller (Thanks to Christopher Jones and Stefano Rivera)
Author-email: p@pwaller.net
License: MIT
Location: /Users/haoliangma/.local/lib/python3.10/site-packages
Requires: 
Required-by: 
```

从这里可以，`pip list` 中列出的`pyfiglet`，是安装在用户目录的 site-packages 中。

查看 macOS 系统自带的 Python 安装了哪些包：

```bash
❯ pip3 list
Package      Version
------------ --------
altgraph     0.17.2
future       0.18.2
macholib     1.15.2
pip          21.2.4
setuptools   58.0.4
six          1.15.0
vboxapi      1.0
wheel        0.37.0
```

这里的列表通常会长很多，因为它包含了很多操作系统或开发工具（如 Xcode）正常运行所依赖的包。

在一个全新的、刚刚激活的虚拟环境中执行 `pip list`：

```bash
❯ pip3 list
Package    Version
---------- -------
pip        23.0.1
setuptools 65.5.0
```

### `pip freeze`

`pip list` 非常适合日常查看，但当我们需要将项目依赖分享给他人或进行部署时，`pip freeze` 命令是更好的选择。

`pip freeze` 的输出格式与 `pip list` 类似，但它遵循一种特定的格式，可以直接用于依赖文件。

```bash
❯ pip freeze > requirements.txt
```

### `pip install`

使用依赖文件重建环境：

```bash
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $ pip install -r requirements.txt
```

## 总结

本文从一个常见的认知误区出发，首先明确了 `pip` 本身是依赖于 `python` 解释器运行的一个包。

随后，我们深入探讨了在复杂的开发环境中，如何使用命令，定位当前正在使用的 `python` 与 `pip` 的真实位置，详细解析了 `pip install` 的三大目的地：全局、用户目录和虚拟环境。

为了方便回顾和查阅，以下是本文提到的所有核心命令及其功能摘要：

| 功能分类 | 命令 | 描述 |
| :--- | :--- | :--- |
| **定位与识别** | `which <command>` | (macOS/Linux) 在 `PATH` 环境变量中查找并显示命令的可执行文件路径。 |
| | `where <command>` | (Windows) 功能与 `which` 类似，在 `PATH` 中查找命令的路径。 |
| | `python3 -c "import sys; print(sys.executable)"` | **精准查找**并打印当前正在运行的 Python 解释器的绝对路径，不受 `shims` 等影响。 |
| | `pip3 --version` | 显示 `pip` 的版本号及其所属的 Python 环境和具体的 `site-packages` 路径。 |
| | `pip3 show <package>` | 显示指定包的详细信息，尤其是其 `Location` 字段，是**确认包安装位置**的关键命令。 |
| | `python3 -c "import site; print(site.getsitepackages())"` | 显示当前 Python 环境的 `site-packages` 目录列表。 |
| | `python3 -m site --user-site` | 显示当前 Python 环境的**用户专属** `site-packages` 目录路径。 |
| **包管理** | `pip3 install <package>` | 安装一个 Python 包。默认尝试全局安装，无权限时可能自动转为用户目录安装。 |
| | `pip3 install --user <package>` | 明确指定将包安装到**用户** `site-packages` 目录。 |
| | `pip3 install -r requirements.txt` | 从指定的 `requirements.txt` 文件中读取并安装所有依赖包，用于环境复现。 |
| | `pip3 list` | 列出当前环境中所有已安装的包及其版本，适合日常查看。 |
| | `pip3 freeze > requirements.txt` | 将 `pip freeze` 的输出重定向，生成或覆盖 `requirements.txt` 依赖清单文件。 |
