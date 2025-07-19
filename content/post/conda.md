---
title: "一文彻底搞懂 Python 环境管理神器 Conda"
date: 2025-07-19T17:24:32+08:00
draft: false
tags: [python]
categories: [tech]
---
## 为什么需要环境管理？

“在我电脑上明明能跑啊！”

这句经典的抱怨背后，是无数开发者都曾遭遇的“依赖地狱”：项目 A 需要 `TensorFlow 1.x`，新项目 B 却要 `TensorFlow 2.x`；团队成员间的环境难以统一；系统自带的 Python 环境被各种库弄得一团糟。

解决这一切的关键在于**环境隔离**：为每个项目创建一个独立的、干净的“沙盒”。而 **Conda**，正是实现这一目标的利器。

本文将是一份全面的 Conda 指南，带你从核心用法、工作原理到最佳实践，让你彻底告别环境管理的混乱，拥抱一个更专业、高效的开发工作流。

## Conda 是什么？

很多初学者会将 Conda 简单地等同于一个虚拟环境工具，就像 Python 自带的 `venv`。这其实只说对了一半。要真正理解 Conda，你需要认识它的三重身份：

1. **环境管理器 (Environment Manager)**

这是 Conda 最广为人知的功能。它允许你创建相互隔离的独立环境。每个环境都可以拥有自己专属的 Python 版本和一套独立的软件包。

*   你可以为项目 A 创建一个搭载 Python 3.7 和旧版库的环境。
*   同时为项目 B 创建另一个搭载 Python 3.10 和最新库的环境。

这两个环境井水不犯河水，你可以通过一条简单的命令在它们之间自由切换。

2. **包管理器 (Package Manager)**

这是 Conda 与 `venv` + `pip` 组合的一个区别：`pip` 是 Python 官方的包管理器，它主要从 PyPI (Python Package Index) 下载软件包；而 Conda 拥有自己独立的包管理系统和软件源（Channels）。

它的关键优势在于：

*   **跨语言支持：** Conda 不仅仅能安装 Python 包！它可以安装和管理任何语言的软件包，比如 C/C++ 库、R 语言包、CUDA 工具链、MKL 数学库等。这对于数据科学和机器学习领域至关重要，因为许多高性能计算库（如 NumPy, TensorFlow）的底层都依赖于这些非 Python 组件。Conda 会一并帮你处理好这些复杂的依赖关系。

*   **二进制分发：** `pip` 有时会下载源码包，需要在你的本地机器上进行编译，这个过程可能因为缺少编译器或依赖库而失败。而 Conda 官方渠道中的包绝大多数都是预先编译好的二进制文件，针对你的操作系统（Windows, macOS, Linux）量身打造。这意味着 `conda install` 通常比 `pip install` 更快、更稳定，极大地避免了烦人的编译错误。

3. **发行版管理器 (Distribution Manager)**

这一点常常被忽视。Conda 能够将 **Python 解释器本身**也视作一个普通的软件包来管理。当你执行 `conda create -n myenv python=3.9` 时，Conda 不会去寻找你系统上已有的 Python，而是会从自己的软件源中下载一个纯净、独立的 Python 3.9 解释器，并安装到 `myenv` 这个环境中。

这意味着，你无需借助 `pyenv` 这类工具，只用 Conda 就能在同一台机器上轻松拥有和管理 Python 3.7, 3.8, 3.9... 等任意多个版本，并将它们分配给不同的项目环境。

**总结一下：** `venv` 只负责创建隔离的环境“空壳”，包的安装和管理仍由 `pip` 负责，且它无法管理 Python 版本。而 Conda 则是一个**集环境隔离、包安装、依赖处理、Python 版本管理于一身的“全能瑞士军刀”**。

## 下载和安装 

### Anaconda vs. Miniconda

在安装 Conda 时，你通常会遇到两个选项：Anaconda 和 Miniconda。这常常让新手感到困惑。

可以这样理解：

*   **Anaconda：** 这是一个“精装修豪华套餐”。它不仅包含了核心的 Conda 工具，还预装了一个特定版本的 Python 和超过 150 个常用的科学计算、数据分析包，如 NumPy, Pandas, Scipy, Jupyter Notebook 等。它的安装包体积较大（通常几百 MB 到数 GB），旨在提供“开箱即用”的体验。

*   **Miniconda：** 这是一个“毛坯房”。它只包含了最核心的 Conda 工具和一个基础的 Python 解释器。整个安装包非常小巧（几十 MB）。安装完成后，你的环境是几乎纯净的，里面没有任何多余的包。你需要什么，就通过 `conda install` 自己动手安装什么。

**我的建议是：**

> **对于所有开发者，尤其是追求环境纯净和良好习惯的开发者，我们强烈推荐从 Miniconda 开始。**

安装 Miniconda 后，你就拥有了完整的 Conda 功能。需要用到 Anaconda 里的那些包？没问题，只需创建一个新环境，然后用 `conda install numpy pandas jupyter` 一行命令就能搞定，效果完全一样。

### 下载 Miniconda

准备好开始了吗？让我们一起动手安装 Miniconda。

访问 anaconda 的官方下载页面[https://www.anaconda.com/download](https://www.anaconda.com/download)，点击跳过注册，进入正式下载页面。

![anaconda](https://cdn.mahaoliang.tech/2024/202507191817954.png)

在页面右侧“Miniconda Installers”的下方，根据你的操作系统和芯片架构，选择最新的安装包。

![Miniconda](https://cdn.mahaoliang.tech/2024/202507191822829.png)

### 安装 Miniconda

* **Windows 用户**
1. 双击下载的 `.exe` 文件。
2. 在 "Installation Type" 步骤，推荐选择 **“Just Me”**，这可以避免很多权限问题。
3. **关键步骤：** 安装程序会提供两个高级选项，请按照推荐设置：
    *   **不勾选** "Add Miniconda3 to my PATH environment variable" (将 Miniconda 添加到系统 PATH)。官方不推荐这样做，因为它可能干扰系统上其他的软件。我们应当使用 Conda 自己的方式来激活环境。
    *   **勾选** "Register Miniconda3 as my default Python" (将 Miniconda 注册为默认 Python)。如果你希望系统默认的 Python 就是 Conda 的，可以勾选，**但对于初学者，不勾选也无妨，保持系统纯净**。
4. 完成安装后，通过“开始菜单”找到并打开 **Anaconda Prompt (Miniconda3)** 来使用 Conda。

* **macOS / Linux 用户**
1. 打开你的终端 (Terminal)。
2. 进入到你下载 `.sh` 文件的目录（通常是 `Downloads` 目录）。
3. 运行安装脚本，命令如下（请将文件名替换为你下载的实际文件名）：
    ```bash
    bash Miniconda3-latest-MacOSX-arm64.sh
    ```
4. 安装过程中，按 `Enter` 查看许可协议，然后输入 `yes` 同意。
5. 当询问安装位置时，直接按 `Enter` 接受默认路径即可（通常是 `~/miniconda3`）。
6.  **关键步骤：** 安装的最后，它会询问 `Do you wish to update your shell profile to automatically initialize conda?`。**务必输入 `yes`**。这一步会自动修改你的 shell 配置文件（如 `.bashrc` 或 `.zshrc`），让 `conda` 命令在你的命令行终端中可用。

完成安装后，**请务必关闭并重新打开你的终端窗口**（或 Anaconda Prompt）。这是为了让刚才的 `conda init` 配置生效。

然后，输入以下命令：

```bash
conda --version
```

如果安装成功，它会显示出 Conda 的版本号，例如 `conda 25.5.1`。你可能还会注意到，命令行提示符的前面多了一个 `(base)` 的字样。这表示你当前正处于 Conda 的 `base` 环境中。

恭喜你，Conda 已经成功安装并准备就绪！接下来，我们将学习如何驾驭它。

## Conda 核心命令实战

Conda 的强大之处在于其简洁而强大的命令行接口。掌握下面这些核心命令，你就足以应对 95% 以上的日常开发需求。

打开你的终端（macOS/Linux）或 Anaconda Prompt (Windows)，让我们开始施展魔法。

### 环境管理 (Environment Management)

环境管理是 Conda 的基石。请记住：**不要在 `base` 环境中工作，为每个项目创建新环境。**

* **创建新环境**

假设我们要启动一个名为 `data_analysis` 的新项目，并且希望使用 `Python 3.10.18`。

```bash
# 语法: conda create --name <环境名> python=<python版本> [其他包...]
conda create --name data_analysis python=3.10.18
```

Conda 会解析依赖，下载一个独立的 Python 3.10.18 和一些基础包，然后为你创建一个名为 `data_analysis` 的环境。你还可以在创建时就指定要安装的包：

```bash
# 创建环境的同时安装 pandas 和 matplotlib
conda create -n data_analysis python=3.10.18 pandas matplotlib
```

注意，参数 `--name` 可以缩写为 `-n`。

* **查看可安装的 Python 版本**

在创建环境时，你可能会问：“我怎么知道哪些 Python 版本是可用的呢？”Conda 把 Python 也当作一个包来管理，所以我们可以用 `search` 命令来查找：

```bash
conda search python
```

这个命令会列出 Conda 软件源中所有可供安装的 Python 版本。这样，在执行 `conda create` 之前，你就可以清楚地知道有哪些版本可以选择。

* **激活（进入）环境**

环境创建好后，它就像一个独立的房间，使用 `activate` 命令进入它：

```bash
conda activate data_analysis
```

执行后，你会发现命令行提示符的前缀从 `(base)` 变成了 `(data_analysis)`。这明确地告诉你：**你现在就在 `data_analysis` 这个沙盒里了！** 在此之后，你所有关于 `python`、`pip`、`conda install` 的操作，都将只影响这个环境。

* **停用（退出）环境**

项目工作完成后，或者需要切换到其他项目时，使用 `deactivate` 命令退出当前环境，返回到 `base` 环境。

```bash
conda deactivate
```

执行后，提示符前面的 `(data_analysis)` 就会消失，变回 `(base)`。

* **列出所有环境**

想看看自己都创建了哪些独立环境？

```bash
conda env list
# 或者 conda info --envs
```

这个命令会列出所有已创建的环境，并在当前激活的环境旁边用星号 `*` 标记。

*   **删除环境**

当一个项目彻底结束，不再需要对应的环境时，可以将其彻底删除以释放磁盘空间。

```bash
# 确保你已退出该环境 (conda deactivate)
# 语法：conda remove --name <环境名> --all
conda remove -n data_analysis --all
```

`--all` 参数至关重要，它会删除环境下的所有包以及环境本身。

### 包管理 (Package Management)

进入了指定的环境后，接下来就是为项目“添砖加瓦”——安装所需要的各种库。

* **安装包**

在**已激活**的环境中，使用 `conda install` 命令。

```bash
# 激活环境
conda activate data_analysis

# 安装单个包
conda install scikit-learn

# 同时安装多个包
conda install beautifulsoup4 requests

# 安装指定版本的包
conda install tensorflow=2.10.0
```

* **查看已安装的包**

```bash
# 必须在激活的环境中执行
conda list
```

上面的命令会列出当前环境中所有的包及其版本号。

* **搜索可用的包**

不确定某个包是否存在，或者想看看有哪些可用的版本？

```bash
conda search numpy
```

这个命令会搜索所有名为 numpy 的包及其可用版本。

* **更新包**

```bash
# 更新单个包到最新兼容版本
conda update pandas

# 更新环境中的所有包
conda update --all 
```

* **删除包**

```bash
# 从当前环境中删除一个包
conda remove beautifulsoup4
```

### **环境复现**

这是 Conda 最强大的功能之一，也是保证团队协作一致性的关键。我们可以将一个环境的所有配置导出到一个文件中，其他人拿到这个文件就能一键复制出完全相同的环境。

* **导出环境配置**

首先，激活你想要导出的环境，然后执行：

```bash
conda activate data_analysis
conda env export > environment.yml
```
    
这会生成一个名为 `environment.yml` 的文件。打开它看看，里面精确记录了环境名称、所有包的版本号。

* **从文件创建环境**

当你的同事拿到这个 `environment.yml` 文件后，只需一行命令，就能在自己的机器上克隆出一个一模一样的环境：

```bash
# 无需先创建环境，Conda 会根据文件中的名字自动创建
conda env create -f environment.yml
```

### 核心命令总结

| 功能分类     | 常用命令                                        | 说明                                                              |
| :----------- | :---------------------------------------------- | :---------------------------------------------------------------- |
| **环境管理** | `conda create -n myenv python=3.10.18`              | 创建一个名为 myenv 的新环境，并指定 Python 版本。                 |
|              | `conda activate myenv`                          | 激活（进入）指定的环境。                                          |
|              | `conda deactivate`                              | 停用（退出）当前环境，返回 base 环境。                            |
|              | `conda env list`                                | 列出所有已创建的环境。                                            |
|              | `conda remove -n myenv --all`                   | 彻底删除一个环境及其所有内容。                                    |
| **包管理**   | `conda install <package_name>`                  | 在当前激活的环境中安装一个或多个包。                              |
|              | `conda list`                                    | 查看当前激活环境中已安装的所有包。                                |
|              | `conda update <package_name>`                   | 更新指定的包。                                                    |
|              | `conda remove <package_name>`                   | 从当前激活的环境中移除一个包。                                    |
| **环境复现** | `conda env export > environment.yml`            | 将当前激活环境的配置导出到 `environment.yml` 文件。               |
|              | `conda env create -f environment.yml`           | 根据 `environment.yml` 文件创建或复现一个完整的环境。             |
| **信息查询** | `conda --version`                               | 查看 Conda 的版本号。                                             |
|              | `conda search python`                           | 列出 Conda 源中所有可供安装的 Python 版本。                       |
|              | `conda search <package_name>`                   | 在软件源中搜索一个包的所有可用版本。                              |

## 深入原理：Conda 是如何工作的？

我们已经学会了 Conda 的核心命令，能够熟练地创建、切换和管理环境。但是，你是否好奇过：

*   当执行 `conda install numpy` 时，这个包到底被安装到哪儿去了？
*   `conda activate myenv` 这条命令，是如何让 `python` 指向一个全新的解释器？
*   Conda 会不会搞乱系统自带的 Python 环境呢？

让我们一起深入 Conda 的“后台”，一探究竟。

### **依赖安装到哪里了？**

当安装 Miniconda 后，它会在你的用户主目录下创建一个文件夹：macOS 上默认是`~/miniconda3`，Windows 上默认是 `C:\Users\<用户名>\miniconda3`。这个文件夹就是 Conda 的大本营，其中有两个子目录至关重要：`envs` 和 `pkgs`。

* **`envs` 目录：环境的独立“公寓”**

`envs` 目录是所有虚拟环境的家。当你执行 `conda create -n data_analysis` 时，Conda 会在 `envs` 目录下创建一个名为 `data_analysis` 的子文件夹。

这个 `data_analysis` 文件夹**不是一个空壳，而是一个几近完整的、独立的 Python 环境**。它里面包含了：

* 一个独立的 Python 解释器 (`envs/data_analysis/bin/python`)
* 独立的包安装目录 (`envs/data_analysis/lib/pythonX.X/site-packages`)
* 所有安装到这个环境的包的二进制文件和脚本 (`envs/data_analysis/bin/`)

```bash
❯ tree -L 3 ~/miniconda3/envs
/Users/haoliangma/miniconda3/envs
└── data_analysis
    ├── bin
    │   ├── ...
    │   ├── python -> python3.10
    │   ├── python3 -> python3.10
    │   ├── python3.1 -> python3.10
    │   ├── python3.10
    │   └── ...
    ├── ...
    ├── lib
    │   ├── ...
    │   ├── python3.10
    │   │   ├── site-packages
    │   │   │── ...
    │   │   └── ...
    │   └── ...
    └── ...
```

每个环境都是一个独立的目录，这就是 Conda 实现文件级别隔离的基础。删除环境时，只需删掉 `envs` 下对应的文件夹即可，干净利落。

* **`pkgs` 目录：包的中央仓库**

这个目录存储所有下载的包文件缓存。Conda 会将下载的包保存在这里，以便在创建或更新环境时可以快速访问，而无需重新下载。

### 如何做到隔离的？

文件隔离解决了存储问题，但运行时隔离又是如何实现的呢？为什么在激活 `data_analysis` 后，我在终端里输入 `python`，系统就知道要运行 `~/miniconda3/envs/data_analysis/bin/python`，而不是系统自带的 `/usr/bin/python`？

答案在于一个至关重要的环境变量：`PATH`。

`PATH` 变量是一系列由冒号（在 Windows 上是分号）隔开的目录路径。当你在终端输入一个命令（如 `python`、`pip`）时，操作系统会**按照 `PATH` 变量中列出的顺序**，从左到右依次在这些目录里查找是否存在同名的可执行文件。一旦找到，就立即执行，并停止向后搜索。

`conda activate data_analysis` 命令的核心魔法就在于：

> **它会暂时性地修改当前终端会话的 `PATH` 变量，将当前激活环境的 `bin` 目录路径（例如 `~/miniconda3/envs/data_analysis/bin`）添加到 `PATH` 变量的最前面。**

让我们看一个例子：

*   **激活前**，查看 `PATH` 环境变量：

```bash
$ echo $PATH
/Users/haoliangma/miniconda3/bin:...
```

*   执行 `conda activate data_analysis` **之后**，查看 `PATH` 环境变量：

```bash
$ conda activate data_analysis
$ echo $PATH
/Users/haoliangma/miniconda3/envs/data_analysis/bin:...
```

现在，当你输入 `python`，操作系统会首先在 `~/miniconda3/envs/data_analysis/bin` 目录里查找。找到了，它立刻执行这个文件，搜索结束。

系统自带的 `/usr/bin/python` 因为排在后面，根本没有机会被找到。这就实现了运行时的完美隔离。

而 `conda deactivate` 命令则执行相反的操作：它会从 `PATH` 变量中移除之前添加的路径，将其恢复到激活前的状态。

### Conda 会影响系统环境吗？

设计上，Conda 不会“污染”你的系统环境。

Conda 所有的环境和包都严格限制在 Miniconda 的安装目录内。`conda activate` 对 `PATH` 的修改也仅限于当前的终端会话，一旦关闭窗口，一切都会复原。它不会去修改或覆盖你系统目录（如 `/usr/bin`）下的任何文件。

**唯一的“例外”是 `conda init` 命令。**

在你安装 Miniconda 的最后一步，或者手动执行 `conda init` 时，它会在你的 Shell 配置文件（如 `~/.bashrc`, `~/.zshrc`）的末尾添加一小段脚本。

**这一步是必要且安全的。** 这段脚本的作用是让 `conda activate` 等命令能够正确地修改你当前 Shell 的环境变量。如果没有它，`conda` 命令本身可能可用，但 `activate` 这种需要与 Shell 交互的功能将无法工作。

你可以随时打开你的 `.bashrc` 或 `.zshrc` 文件查看 Conda 添加的内容，它有清晰的注释 `# >>> conda initialize >>>`。如果你想彻底移除 Conda，除了删除 Miniconda 的安装目录，只需将这段脚本从配置文件中删除即可。

## 横向对比，选择适合的工具

Conda 功能强大，但它并非唯一的环境管理工具。在 Python 的世界里，还有 `venv` 和 `pyenv` 等广受欢迎的工具，它们有各自不同的适用场景。

理解它们之间的区别，能帮助你根据自己的项目需求，选择最合适的工具。

### Conda vs. venv

这是最常见的比较。`venv` 是 Python 3.3+ 版本中内置的虚拟环境创建工具，它通常与 `pip`（Python 官方包安装器）配合使用，形成了一套轻量级的环境管理方案。

让我们用一个表格来清晰地对比它们：

| 特性 / 对比项 | Conda | venv + Pip |
| :--- | :--- | :--- |
| **核心定位** | **一体化解决方案**：环境、包、Python 版本全包 | **轻量级组合拳**：venv 管环境，pip 管包 |
| **管理范围** | **语言无关**：能管理 Python, R, C++, CUDA 等任何软件包。 | **专注 Python**：只能管理 Python 包。 |
| **Python 版本管理** | **内置功能**：可自行下载和管理任意版本的 Python 解释器。 | **不具备**：使用创建环境时系统当前激活的 Python 版本。 |
| **包来源** | Conda Channels (如 anaconda, conda-forge)，主要是**预编译二进制包**。 | PyPI (Python Package Index)，包含源码包和 Wheels (二进制包)。 |
| **依赖解析能力** | **非常强大**。Conda 在安装前会进行严格的依赖关系检查，能解决复杂的非 Python 依赖（如 MKL, cuDNN）。 | **相对较弱**。Pip 的依赖解析器近年来有改进，但处理复杂或冲突的依赖时仍可能遇到困难。 |
| **跨平台一致性** | **极高**。通过 `environment.yml` 文件，可以保证在 Windows, macOS, Linux 上复现几乎完全一致的环境。 | **较好，但有风险**。`requirements.txt` 文件在不同操作系统间可能因系统级依赖不同而表现不一。 |
| **适用场景** | **数据科学、机器学习、生物信息学**等需要复杂**非 Python 依赖**的项目；需要管理多 Python 版本的场景。 | **Web 开发 (Django, Flask)、纯 Python 库开发、简单脚本**等依赖相对纯净的场景。 |

**简单总结：**

* **选择 `venv + pip`**，如果你：
    * 在做纯粹的 Python 项目，如 Web 开发。
    * 项目的依赖项简单，不涉及复杂的 C/C++ 库。

* **选择 `Conda`**，如果你：
    * 数据科学或科学计算领域，需要处理 NumPy, SciPy, TensorFlow 等依赖复杂底层库的包。
    * 需要在一个项目中混合使用 Python 和其他语言的工具（如 R）。
    * 希望一个工具就能搞定环境和 Python 版本的所有问题。

### Conda vs. pyenv

另一个让初学者困惑的组合是 `Conda` 和 `pyenv`。两者似乎都能管理 Python 版本，它们有什么不同？

`pyenv` 是一个**纯粹的 Python 版本管理器**。它的目标只有一个：让你在系统上轻松安装、切换多个 Python 版本（例如，全局用 3.10，某个项目用 3.8）。它本身**不管理虚拟环境**。`pyenv` 需要和 `venv` 配合使用：用 `pyenv` 切换到项目的目标 Python 版本，然后用该版本下的 `venv` 模块创建项目专属的虚拟环境。

`pyenv` 通过一种名为 "shims" 的机制工作。它会在你的 `PATH` 路径最前面插入一个 `~/.pyenv/shims` 目录。当你执行 `python` 命令时，实际运行的是 `shims` 里的一个脚本，这个脚本会根据你当前的配置（全局、项目局部等）决定启动哪个版本的真实 Python 解释器。

如前所述，Conda 将 Python 解释器本身也视为一个普通的“包”。`conda create -n data_analysis python=3.8` 会为你下载并安装一个独立的 Python 3.8，它与系统中的其他 Python 绝缘。

**关键区别：** `pyenv` 管理的是“裸露”的 Python 解释器，而 `Conda` 管理的是包含 Python 解释器的“环境”。

#### 强烈建议不要同时使用 Conda 和 pyenv

混用 Conda 和 pyenv 就像让两个不同的交通指挥员，在同一个十字路口同时指挥交通，结果必然是**混乱和冲突**。

**冲突的根源在于 `PATH` 的控制权之争：**

*   `pyenv` 通过 shims 机制，劫持了 `python` 命令的调用。
*   `Conda` 通过 `conda activate`，修改 `PATH` 变量，也想控制 `python` 命令的指向。

当你同时安装并初始化了两者，执行 `python` 命令时，到底听谁的？这取决于你的 Shell 配置中，谁的初始化脚本排在后面，谁就可能覆盖前者的设置。这会导致一些极其诡异且难以排查的问题。

如果你的工作流**以数据科学为中心**，或者你喜欢 Conda 的一体化便利性，**那就只用 Conda**。让它来管理你所有的环境和 Python 版本。

如果你是一名 **Python Web 开发者**或库开发者，偏爱 **UNIX-like 的小工具组合哲学**，那么 **`pyenv + venv`** 是一个非常优雅和强大的组合。

避免将两者混合，可以为你节省大量调试环境问题的时间，让你可以专注于代码本身。

## 总结

回顾一下我们的收获，Conda 的核心价值：

*   **不仅仅是虚拟环境：** 我们了解到，Conda 是一个集**环境管理、包管理、Python 版本管理**于一身的“三合一”强大工具。它通过在 `envs` 目录中创建隔离的文件系统，并在 `pkgs` 目录中共享缓存，实现了高效、节省空间的隔离。

*   **强大的依赖处理：** 凭借其跨语言的包管理能力和预编译的二进制包，Conda 能够轻松处理那些依赖复杂底层库（如 C++, FORTRAN, CUDA）的科学计算包，这是它在数据科学领域封神的关键。

我们对比了 Conda 与 venv/pip 和 pyenv 的区别，结论是：没有最好的工具，只有最适合你当前场景的工具。

最后，我想分享我个人的选择策略，希望能给你提供一个更具体的参考：

* **在 macOS 上，当我进行纯 Python 开发（如 Web 后端、爬虫脚本）时，我倾向于使用 `pyenv` + `venv` 的组合。** `pyenv` 负责管理和切换全局的 Python 版本，`venv` 则为每个项目创建极致轻量的虚拟环境。这套组合非常优雅，工具链清晰解耦，完全符合这类项目的需求。

* **在我的 Windows 笔记本上，由于配备了 NVIDIA 显卡，主要用于 AI 和机器学习开发，我则毫不犹豫地选择 Conda。** 原因很简单：AI 开发常常依赖于复杂的 CUDA 和 cuDNN 工具链。在 Windows 平台上，手动配置这些非 Python 依赖是一件非常痛苦的事，而 Conda 可以通过 `conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia` 这样的一行命令，完美地处理好所有依赖关系，这几乎是不可替代的便利。
