---
title: "现代 Python 项目管理指南"
date: 2025-07-23T18:39:02+08:00
draft: false
tags: [python]
categories: [tech]
---
在很长一段时间里，我管理 Python 项目的方式堪称“原始”。每当项目需要新的依赖，我便会在 `README.md` 文件里手动记上一笔，提醒自己需要 `pip install` 哪些库。这种“刀耕火种”的模式，对于个人写的一些小脚本尚可应付，但当项目变得复杂，或是需要与他人协作时，其脆弱和低效便暴露无遗。

真正的转折点发生在今年夏天。我有幸参与了[开源之夏](https://summer-ospp.ac.cn/)，并成功中选了 openEuler 社区的 [面向 openEuler distroless 镜像的 SDF 自动生成工具开发](https://summer-ospp.ac.cn/org/prodetail/25b970448) 项目，为 [splitter](https://gitee.com/openeuler/splitter) 这个工具贡献代码。当我满怀激情地克隆代码库，准备大展拳脚时，一个文件赫然出现在我的眼前：`pyproject.toml`。这个我以往只是模糊听闻过的文件，在这里却是项目配置的核心。

这次经历像一扇窗，让我窥见了现代 Python 项目管理的全新世界。于是，我将这段学习和探索的经历整理成文。

## 传统方式：`venv` + `requirements.txt`

以 `venv` 和 `requirements.txt` 为核心的工作流，是 Python 社区在告别全局安装、走向规范化管理过程中迈出的重要一步。

### 环境隔离与依赖列表

**虚拟环境 (`venv`)**

如果你经历过在系统全局 Python 环境中安装各种包的混乱时期，你一定对 **“依赖地狱” (dependency hell)** 这个词深有体会。项目 A 需要 `requests==2.20.0`，而项目 B 依赖的另一个库却需要 `requests==2.28.0`，它们在全局环境中相互冲突，让开发者头痛不已。

`venv` 的出现正是为了解决这个问题。它允许我们为每个项目创建一个独立的、与全局环境隔离的 Python 工作空间。

```bash
# 在项目根目录下，创建一个名为 .venv 的虚拟环境
python3 -m venv .venv
```

这个命令会创建一个 `.venv` 文件夹，里面包含了项目所需的一个迷你的 Python 运行环境。要使用它，我们需要先“激活”：

```bash
source .venv/bin/activate
```

激活后，你会发现命令行提示符前面多了 `(.venv)` 的标识。此时，所有 `pip` 的安装、卸载操作都将被限制在这个独立的虚拟环境中，再也不会污染全局环境了。

**依赖文件 (`requirements.txt`)**

环境隔离了，但如何与他人协作呢？我们总不能把整个 `.venv` 文件夹都发给别人吧。这时，`requirements.txt` 就登上了历史舞台。它的作用，就是一份项目的“依赖清单”。

最常见的生成方式是使用 `pip freeze` 命令：

```bash
# 将当前虚拟环境中所有已安装的包及其精确版本号导出
pip freeze > requirements.txt
```

这样，你的项目协作者或者部署服务器，只需要拿到你的代码和这个 `requirements.txt` 文件，然后执行一条简单的命令，就可以复现出一个一模一样的运行环境：

```bash
# 从 requirements.txt 文件中安装所有指定的依赖
pip install -r requirements.txt
```

这套组合拳极大地提升了 Python 项目的规范性和可复现性，至今仍有大量项目在使用。

### 传统方式的隐患

尽管 `venv` + `requirements.txt` 解决了大问题，但随着项目复杂度的提升，其内在的缺陷也逐渐暴露出来。

**依赖混淆**

最大的问题在于 `pip freeze` 的工作方式。它像一个不加分辨的记录员，会把你环境中**所有**的包都记录下来。这其中既包含了你为了实现功能而主动安装的**直接依赖**（比如 Web 框架 `Flask`），也包含了 `Flask` 运行所必需的**间接依赖**（比如 `Werkzeug`, `Jinja2`, `click` 等）。

最终生成的 `requirements.txt` 文件看起来会是这样：

```
# requirements.txt
blinker==1.7.0
click==8.1.7
Flask==3.0.0
itsdangerous==2.1.2
Jinja2==3.1.3
MarkupSafe==2.1.3
Werkzeug==3.0.1
```

在这个文件里，你已经分不清谁是谁的依赖了。项目的核心依赖关系被模糊掉了，给后续的维护，比如升级某个特定的核心库，带来了不小的麻烦。

**孤儿依赖**

另一个令人头疼的问题是“孤儿依赖”。假设你的项目后续不再需要 `Flask` 了，于是你执行了 `pip uninstall flask`。`pip` 很听话地卸载了 `Flask` 本身，但它当初为了 `Flask` 而自动安装的 `blinker`、`click` 等间接依赖，却被遗留在了环境中，变成了无人认领的“孤儿”。

日积月累，你的虚拟环境会因为这些残留的孤儿依赖而变得越来越臃肿，还可能在未来引发难以预料的依赖冲突。

正是因为这些的缺陷，Python 社区开始探索一种更清晰、更智能的管理方案。这便引出了我们下一章的主角——`pyproject.toml`。

## 现代篇章：`pyproject.toml`

面对 `requirements.txt` 带来的依赖混淆问题，Python 社区需要一个更强大、更规范的解决方案，答案就是 `pyproject.toml`。

### 标准的诞生 (PEP 518 & 621)

在 `pyproject.toml` 成为标准之前，一个 Python 项目的配置信息可谓“四分五裂”。项目元数据可能在 `setup.py` 或 `setup.cfg` 里，运行依赖在 `requirements.txt` 中，测试配置在 `tox.ini` 或 `.coveragerc` 里，代码格式化工具 `black` 和静态检查工具 `mypy` 又有它们各自的配置文件。这种碎片化的状态让项目维护变得异常繁琐。

[PEP 518](https://peps.python.org/pep-0518/) 的提出正是为了终结这种乱象。它定义了一个名为 `pyproject.toml` 的文件格式，旨在成为**所有构建工具的统一配置入口**。随后，[PEP 621](https://peps.python.org/pep-0621/) 进一步规范了如何在这个文件中声明项目的核心元数据（如名称、版本、作者和依赖项），使其彻底摆脱了对 `setup.py` 的依赖。

简单来说，`pyproject.toml` 的使命就是将所有与项目相关的配置，集中到一个官方认可的、格式统一的文件中。

### 声明式依赖管理

`pyproject.toml` 最重要的改进，在于它引入了**声明式依赖管理**。与 `pip freeze` 那种不加区分的全量记录不同，我们现在只需要在 `pyproject.toml` 文件中清晰地声明项目的**直接依赖**。

让我们回到上一章的 `Flask` 项目，它的 `pyproject.toml` 文件现在会是这样：

```toml
# pyproject.toml
[project]
name = "my-flask-app"
version = "0.1.0"
dependencies = [
  "Flask==3.0.0" 
]
```

看，多么清爽！`dependencies` 列表里只有 `Flask`。我们在这里表达的是**意图**：“我的项目需要 Flask 3.0.0 版本”，而不是 `Flask` 运行所需要的所有包的冗长列表。至于 `Flask` 自身依赖的 `Werkzeug`、`Jinja2` 等，将由工具在安装时自动去解析和处理。

这种方式将项目的直接依赖与间接依赖彻底分离，让依赖关系一目了然，极大地提升了项目的可读性和可维护性。

### `pip install .` 的背后原理

现在我们有了 `pyproject.toml`，那么 `pip` 是如何利用它来安装项目的呢？

当你进入项目根目录，在激活的虚拟环境中执行 `pip install .` 背后其实发生了两个关键步骤：构建和安装。这个过程不仅仅是复制文件，而是将你的项目变成一个标准的、可分发的 Python 软件包。

**第一步：构建 (Build)**

`pip` 首先会扮演一个“构建前端”的角色。它会读取 `pyproject.toml` 文件中的 `[build-system]` 表，找到指定的“构建后端”（通常是 `setuptools`）。然后，`pip` 会指示构建后端，依据当前项目的源代码和 `pyproject.toml` 中的元数据，构建出一个标准的 Python 软件包，通常是一个 `.whl` (wheel) 文件。这个 wheel 文件是一个包含了所有代码和元数据的 zip 压缩包，是现代 Python 的标准分发格式。

**第二步：安装 (Install)**

构建完成后，`pip` 会接手这个新鲜出炉的 wheel 文件，并将其内容“解压”并安装到你的虚拟环境中。这个过程会产生以下三类核心产物：

1. **项目代码与元数据 (Project Code and Metadata)**

你的项目代码（即所有的 `.py` 文件和包）会被复制到虚拟环境的 `site-packages` 目录下（例如 `.venv/lib/python3.11/site-packages/my_flask_app`）。

同时，一个名为 `my_flask_app-0.1.0.dist-info` 的元数据目录也会被创建。你可以把它看作是这个软件包的“身份证”，里面包含了从 `pyproject.toml` 中提取的所有信息，比如项目名、版本、作者以及最重要的——依赖列表。

2. **项目依赖 (Project Dependencies)**

`pip` 会读取上述元数据文件，找到其中声明的 `dependencies` 列表（例如 `"Flask==3.0.0"`）。

然后，它会自动下载并安装所有这些直接依赖，以及这些依赖所需要的间接依赖（如 `Werkzeug`, `Jinja2` 等），并将它们全部安装到 `site-packages` 目录中。

3. **可执行的命令行脚本 (Executable Command-line Scripts)**

如果你在文件中定义了 `[project.scripts]` 部分，像这样：

```toml
[project.scripts]
my-app = "my_flask_app.cli:main"
```
那么 `pip` 在安装时，会在虚拟环境的 `bin` 目录（Windows 上是 `Scripts`）下创建一个名为 `my-app` 的可执行文件。

这个文件是一个小小的“启动器”脚本，它的作用是调用当前虚拟环境中的 Python 解释器，并执行你指定的函数 (`my_flask_app.cli:main`)。

正因为如此，一旦安装完成并激活了虚拟环境，你就可以在任何路径下直接通过命令行运行 `my-app` 来启动你的应用程序了。

通过这套标准化的流程，`pip install .` 不仅安装了代码和依赖，还完成了命令行工具的创建，将一个项目从一堆源代码变成了一个功能完整、随时可用的工具。

### 可编辑模式 (`pip install -e .`)

`pip install .` 非常适合用于最终的安装和部署，但在日常开发中，每次修改代码后都重新构建和安装一遍，显然效率太低。为此，`pip` 提供了一种强大的**可编辑模式 (editable mode)**。

```bash
pip install -e .
```

`-e` 参数是这个模式的关键。执行这条命令后，`pip` 不会再把你的项目文件复制到 `site-packages` 目录中，而它会在 `site-packages` 里创建一个特殊的链接文件（`.pth` 文件），这个链接直接指向你当前项目的源代码目录。

这样做的好处是显而易见的：你的项目源代码和虚拟环境中的“已安装版本”实现了实时同步。你在编辑器里对任何 `.py` 文件做的修改，保存后会**立即生效**，无需任何重新安装的步骤。这极大地简化了“修改 - 运行 - 调试”的开发循环，是现代 Python 开发的必备技巧。

通过 `pyproject.toml` 和可编辑模式，我们不仅拥有了清晰的依赖管理，还获得了高效的开发体验。接下来，让我们通过一个真实的项目，来看看 `pyproject.toml` 在实战中是如何发挥作用的。

## 解析`splitter` 项目的 `pyproject.toml`

以我参与的 [splitter](https://gitee.com/openeuler/splitter) 项目为例，解析它的 `pyproject.toml` 文件，看看它是如何将项目的所有配置信息尽收囊中的。

以下是 `splitter` 项目中 `pyproject.toml` 文件的核心内容，为便于说明，已做适当简化：

```toml
# pyproject.toml of splitter project
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "splitter"
version = "1.0.0" # Assuming a version for clarity
description = "A tool for splitting software packages into smaller components."
authors = [{name = "openEuler Cloudnative SIG"}]
readme = "README.md"
requires-python = ">=3.7"
license = {text = "MulanPSL-2.0"}
dependencies = [
    "PyYAML",
    "click",
    "packaging",
    "jinja2"
]

[project.urls]
Homepage = "https://gitee.com/openeuler/splitter"

[project.scripts]
splitter = "tools.main:main"

[tool.setuptools.packages.find]
where = ["."]
exclude = ["tests", "docs"]
```

现在，让我们逐段来剖析这个文件。

### `[build-system]`

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

这是 `pyproject.toml` 的“入口”，也是遵循 PEP 518 规范的体现。它告诉 `pip` 这样的构建工具：在构建本项目之前，请确保你的环境中安装了 `setuptools` 和 `wheel` 这两个包，因为它们是我需要的构建工具。然后，请使用 `setuptools.build_meta` 这个入口点来执行实际的构建操作。

这一段配置，实现了项目构建依赖与项目本身运行依赖的解耦。

### `[project]`

```toml
[project]
name = "splitter"
version = "1.0.0"
description = "..."
authors = [...]
readme = "README.md"
requires-python = ">=3.7"
license = {text = "MulanPSL-2.0"}
dependencies = [
    "PyYAML",
    "click",
    "packaging",
    "jinja2"
]
```

这部分是项目的核心元数据，遵循 PEP 621 规范。

* `name`, `version`, `description`, `authors`：这些都是项目的基本信息，会被打包到软件中，并在 PyPI 等包索引网站上展示。

* `requires-python`：这是一个非常重要的字段，它声明了项目运行所需的最低 Python 版本。如果用户尝试在一个不兼容的 Python 版本（如 Python 3.6）上安装，`pip` 会直接报错并终止安装，避免了后续可能出现的各种运行时错误。

* `dependencies`：这里是项目的依赖。它清晰地列出了 `splitter` 运行所必需的**直接依赖项**。当执行 `pip install .` 时，`pip` 会负责安装 `PyYAML`, `click`, `packaging`, `jinja2` 以及它们各自的所有子依赖，而我们无需关心这些复杂的依赖链。

### `[project.scripts]`

```toml
[project.scripts]
splitter = "tools.main:main"
```

这一行配置，定义了一个名为 `splitter` 的命令行入口。

当项目被安装后，`pip` 会在虚拟环境的 `bin/` 目录下创建一个名为 `splitter` 的可执行文件。当我们运行这个命令时，系统会自动调用 `tools/main.py` 文件中的 `main()` 函数。这使得一个复杂的 Python 项目可以像一个普通的系统命令一样被调用，极大地提升了用户体验。

### `[tool.setuptools.packages.find]`

```toml
[tool.setuptools.packages.find]
where = ["."]
exclude = ["tests", "docs"]
```

前缀为 `[tool.*]` 的表是为各种第三方工具预留的配置空间。这里，我们为构建后端 `setuptools` 提供了配置。

`packages.find` 指示 `setuptools` 自动发现项目中的所有 Python 包。`where = ["."]` 告诉它从当前根目录开始查找，而 `exclude = ["tests", "docs"]` 则明确排除了测试代码和文档目录，确保它们不会被打包到最终发布的应用中。

通过这个真实的例子，我们可以看到 `pyproject.toml` 如何将项目的构建信息、元数据、运行依赖、命令行入口和工具配置，全部地组织在了一起，让 Python 项目的结构变得清晰和标准化。

## Poetry, UV, PDM 等高级管理工具

`venv` + `pyproject.toml` + `pip` 的组合，已经构建起了一个相当稳固和规范的项目管理框架，它解决了依赖隔离和声明的核心问题。然而，这个工作流依然存在一些需要开发者手动操作的环节：

*   **手动管理虚拟环境**：你需要记得先创建，再激活。

*   **手动编辑依赖文件**：添加或移除依赖时，你需要手动去编辑 `pyproject.toml` 文件。

有没有一种工具，能将这些步骤完全自动化呢？当然有。这便是 `Poetry`, `PDM` 以及新秀 `uv` 这类高级项目管理工具的使命所在。

### 新一代高级管理工具

这些工具并非要推翻 `venv` 和 `pyproject.toml`，恰恰相反，它们是建立在这些官方标准之上的**高级封装**和**工作流引擎**。它们的核心理念可以概括为：

1.  **自动化环境管理**：你不再需要关心 `venv` 的创建和激活，工具会自动为你处理好一切。

2.  **命令式依赖操作**：通过简单的命令（如 `add`, `remove`）来管理依赖，工具会自动更新 `pyproject.toml` 文件。

3.  **确定性构建**：通过生成一个精确的 `lock` 文件（如 `poetry.lock`, `pdm.lock`, `uv.lock`），锁定项目中**所有**依赖（包括直接和间接依赖）的精确版本。这确保了任何人在任何时间、任何机器上都能构建出完全一致的运行环境。

4.  **集成化体验**：将依赖管理、环境管理、打包、发布等功能集成到一套统一的命令行接口中，提供“一站式”的解决方案。

### 以 `uv` 为例的现代化工作流

`uv` 是由 `ruff` 的作者开发的最新一代 Python 打包工具，它用 Rust 编写。让我们以 `uv` 为例，体验一下现代化的工作流是多么流畅。

*假设你已经通过 `pip install uv` 安装了它。*

**开发者 A (项目创建者)**

1.  **初始化项目**

在一个空目录中，我们不再需要手动创建任何文件。

```bash
# uv 会引导你创建 pyproject.toml 文件
uv init
```

2.  **添加依赖**

现在，想给项目添加 `flask` 依赖？告别手动编辑，一条命令即可：

```bash
uv add flask
```

这条命令，`uv` 在背后为你完成了一系列操作：

* **检查并创建虚拟环境**：它会自动检测当前目录下是否存在 `.venv`，如果没有，就为你创建一个。

* **修改 `pyproject.toml`**：自动将 `flask` 添加到 `[project.dependencies]` 列表中。

* **解析依赖并安装**：解析 `flask` 的所有依赖树，并将它们全部安装到虚拟环境中。

* **生成锁文件**：创建一个 `uv.lock` 文件，里面精确记录了本次安装的所有包（包括间接依赖）的版本号和哈希值，锁定了当前环境的状态。

**开发者 B (协作者)**

1.  **同步环境**

当开发者 B 从 GitHub 克隆了项目后，他不需要再去研究 `pyproject.toml` 或执行复杂的安装命令。他只需要：

```bash
uv sync
```
`uv` 会读取 `uv.lock` 文件，然后下载并安装所有被锁定的包，为他创建一个与开发者 A **完全一致**的虚拟环境。

**日常开发与执行**

在开发过程中，你甚至不需要手动 `source .venv/bin/activate` 来激活环境。

```bash
# uv 会自动在项目的虚拟环境中执行 python main.py
uv run python main.py
```

`uv run` 命令会自动寻找并使用当前项目的虚拟环境来执行后续的命令，让你的操作更加简洁。

通过 `uv` 的演示，我们可以看到，现代化的项目管理工具将开发者从繁琐的、易出错的手动操作中彻底解放出来。它们通过自动化和确定性的机制，让依赖和环境管理变得简单、可靠且高效。

好的，这是最后一章的总结部分。它将对全文进行回顾，并提炼出具体、可操作的最佳实践，为读者画上一个圆满的句号。

## 构建你的现代化 Python 工作流

无论你正在开始一个新项目，还是打算重构一个旧项目，请遵循以下三个原则。

**1. 隔离是基础：始终为你的项目创建虚拟环境**

这是现代化项目管理的基石，也是最不应该被忽略的一步。为每个项目创建一个独立的虚拟环境，可以从根源上杜绝依赖冲突和全局环境污染。像 `uv`, `Poetry`, `PDM` 这类工具已经将这一步完全自动化，你甚至无需再手动操作。如果你仍在使用原生工具，请务必将 `python -m venv .venv` 作为你开启任何新项目的第一条命令。

**2. 声明是核心：拥抱 `pyproject.toml`，告别 `requirements.txt`**

`pyproject.toml` 是 Python 项目的未来。请将它作为你项目配置的唯一真实来源。

* **只声明直接依赖**：在 `[project.dependencies]` 中，只列出你的项目代码直接 `import` 的那些库。这能让你的项目依赖关系保持最大程度的清晰和可维护性。

* **统一所有配置**：将代码检查工具 (linter)、格式化工具 (formatter)、测试框架等所有工具的配置，都迁移到 `pyproject.toml` 的 `[tool.*]` 表中，让项目根目录保持整洁。

**3. 工具是利器：选择一个现代化的管理工具**

虽然你可以手动维护 `pyproject.toml` 并结合 `pip` 使用，但一个现代化的项目管理工具能极大地提升你的生产力。

* **对于新项目**：强烈推荐直接使用 `uv` 或 `Poetry`。它们提供的 `add`, `remove`, `sync`, `run` 等命令，将依赖管理和环境操作的体验提升到了一个全新的高度。它们带来的**确定性构建**（通过 lock 文件）对于团队协作和持续集成（CI/CD）至关重要。

* **对于现有项目**：迁移到这些工具也比你想象的要简单。大多数工具都提供了从 `requirements.txt` 导入依赖的功能，可以帮助你平滑过渡。

## 写在最后

希望这篇指南能够帮助你理清 Python 项目管理的脉络，并为你提供一套清晰、可行的实践方案。现在就动手，为你的下一个 Python 项目开启一个现代化的新起点吧！