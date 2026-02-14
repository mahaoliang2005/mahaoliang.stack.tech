---
title: "macOS 开发环境管理"
date: 2026-02-13T15:28:25+08:00
draft: false
tags: [macos, python, nodejs, java]
categories: [tech]
---
![macos](https://cdn.mahaoliang.tech/2026/20260214211941723.png)

在 macOS 上进行开发，经常会遇到这样的问题：不同的项目依赖不同版本的 Python、Node.js 或 Java，而 macOS 自带的系统版本往往过时且权限受限。如果直接安装全局版本，很容易弄脏系统环境。

因此，**版本隔离** 和 **工具链管理** 是每个 macOS 开发者的必修课。

本文档旨在作为一份**速查表 (Cheat Sheet)**，汇总了 macOS 上最主流的包管理工具（Homebrew）以及各语言的专用版本管理工具（nvm, uv, sdkman）的常用命令，涵盖安装、版本切换、以及工具自身的维护更新。

## 系统级包管理：Homebrew

Homebrew 是 macOS 的必备工具，主要用于管理系统级工具（如 `wget`, `ffmpeg`）和 GUI 应用（Cask）。

记住，只用它装工具，**不要**用它直接装语言环境（如 Python, Node），否则多版本切换会很麻烦。

### 核心操作：安装与卸载

最基础的增删查改。

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **安装软件** | `brew install <包名>` | 例如 `brew install wget` |
| **安装 App** | `brew install --cask <软件名>` | 例如 `brew install --cask picgo` |
| **卸载软件** | `brew uninstall <包名>` | 删除已安装的包，保留配置文件 |
| **彻底卸载** | `brew uninstall --zap <软件名>` | **(Cask 专用)** 卸载应用并尝试删除相关的配置文件/偏好设置 |
| **查看已安装** | `brew list` | 列出所有安装的包 |
| **搜索软件** | `brew search <关键词>` |  |

### 进阶管理：依赖与清理

随着时间推移，系统中会残留很多不再使用的“孤儿依赖”，这一部分命令用于保持系统干净。

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **查看依赖树** | `brew deps --tree <包名>` | **推荐**。以树状图显示该软件依赖了哪些库，清晰直观。 |
| **查看被谁依赖** | `brew uses --installed <包名>` | 反向查询：查看这个包被谁用到了（防止误删重要组件）。 |
| **自动移除** | `brew autoremove` | **重要**。自动卸载那些“作为依赖被安装但现在不再需要”的孤儿包。 |
| **清理缓存** | `brew cleanup` | 删除旧版本的安装包（.dmg/.tar.gz）和临时文件，释放磁盘空间。 |

### 系统维护：更新与诊断

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **更新 Brew** | `brew update` | 更新 Homebrew 自身和配方库（Formulae）。 |
| **升级软件** | `brew upgrade` | 将已安装的所有软件升级到最新版。 |
| **系统体检** | `brew doctor` | **排错首选**。检查系统环境、权限、路径冲突等问题，并给出修复建议。 |
| **查看信息** | `brew info <包名>` | 查看软件的版本、安装路径、依赖关系及注意事项（Caveats）。 |

## Node.js 管理

参考[官方文档](https://nodejs.org/en/download)，使用 nvm 和 npm 方式安装 Node.js。`nvm` 管理 Node 版本，而 Node 版本内部管理着自己的 `npm` 版本。

### Node.js 版本控制 (nvm)

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **安装版本** | `nvm install <版本号>` | 例如 `nvm install 24` (最新 v24) 或 `nvm install --lts` |
| **切换版本** | `nvm use <版本号>` | 仅在当前终端窗口生效，例如 `nvm use 24` |
| **设置默认** | `nvm alias default <版本号>` | **重要**：设置新开终端时的默认 Node 版本 |
| **查看远程** | `nvm ls-remote` | 列出所有可供安装的 Node 版本 |
| **查看本地** | `nvm ls` | 列出本机已安装的版本 |
| **卸载版本** | `nvm uninstall <版本号>` | 删除不再使用的版本及其全局包 |

### npm 自身管理 (升级与维护)

当你安装 Node 时，它带有一个默认的 npm。但 npm 自身更新频率很高，可以通过手动升级。

**注意：** npm 的升级是**绑定在当前 Node 版本下**的。如果你切换了 Node 版本，npm 版本也会变回那个 Node 自带的版本。

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **升级 npm (最新)** | `npm install -g npm@latest` | **推荐**。将当前 Node 环境下的 npm 升到最新版。 |
| **升级 npm (指定)** | `npm install -g npm@10.9.3` | 降级或锁定特定版本时使用。 |
| **查看版本** | `npm -v` | 查看当前正在使用的 npm 版本。 |

### 全局包管理 (Global Packages)

虽然我们推荐项目依赖都装在本地 (`npm install`)，但某些 CLI 工具 (如 `yarn`, `pnpm`, `typescript`, `nest`) 可能需要全局安装。

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **查看全局包** | `npm list -g --depth 0` | 检查全局装了哪些工具，避免混乱。 |
| **安装全局包** | `npm install -g <包名>` | **无需 sudo**。在 nvm 环境下，权限属于当前用户。 |
| **卸载全局包** | `npm uninstall -g <包名>` |  |
| **迁移全局包** | `nvm install <新版本> --reinstall-packages-from=<旧版本>` | 安装新 Node 时，自动把旧 Node 的全局包也装一份过来。 |

### nvm 自身升级

`nvm` **不支持** `nvm update` 这种命令。升级它本质上就是下载最新的脚本覆盖旧脚本。

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| **升级步骤** | `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh \| bash` | **必须手动指定最新版本号**。运行后重启终端即可。 |
| **检查版本** | `nvm --version` | 确认当前 nvm 的版本。 |

> **注意**：
> 1. 升级 nvm **不会** 删除你已经安装的 Node.js 版本或全局包，放心执行。
> 2. 请务必去 [nvm GitHub](https://github.com/nvm-sh/nvm/releases) 查看最新的版本号（上面的 `v0.40.3` 只是示例）。
> 
> 

## Python 管理：uv

`uv` 是一个用 Rust 编写的 Python 打包和项目管理器，它将多个 Python 工具（如 `pip`、`venv`、`pyenv` 等）的功能整合到一个工具中，提供了一个统一、高效的 Python 开发流程。

我曾经写过一篇技术博客[《Python 项目管理最佳实践：uv 使用指南》](https://mahaoliang.tech/p/python-%E9%A1%B9%E7%9B%AE%E7%AE%A1%E7%90%86%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5uv-%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/)，已经覆盖 `uv` 的常用场景，这里只强调 `uv` 自身的升级：

```bash
# 更新 uv 自身到最新版
uv self update
```

## Java: SDKMAN!

对于 Java 开发者，`SDKMAN!` 是管理 JDK (Java Development Kit) 以及 Maven, Gradle, Kotlin 等 JVM 生态工具的神器。它彻底解决了配置 `JAVA_HOME` 的痛苦。

### 常用命令速查

| 操作 | 命令 | 说明 |
| --- | --- | --- |
| **列出可用版本** | `sdk list java` | 查看所有厂商（Temurin, GraalVM 等）的版本 |
| **安装版本** | `sdk install java 21.0.2-tem` | 安装特定版本 |
| **切换版本 (临时)** | `sdk use java 17.0.10-tem` | 仅当前终端有效 |
| **设置默认版本** | `sdk default java 21.0.2-tem` | 全局永久生效 |
| **查看当前版本** | `sdk current java` |  |
| **查找安装路径** | `sdk home java <版本号>` | 如果 IDE 需要指定 JDK 路径时很有用 |

### 维护与更新

SDKMAN 拥有完善的自更新机制：

```bash
# 1. 更新 SDKMAN 客户端自身
sdk selfupdate

# 2. 更新元数据（获取最新的 Java 版本列表）
sdk update

```

### 总结

在 macOS 上保持环境整洁的秘诀就是：**各司其职**。

* **Brew**: 管系统工具。
* **NVM**: 管 Node.js。
* **uv**: 管 Python。
* **SDKMAN**: 管 Java。

掌握以上命令，基本能覆盖 90% 的日常开发环境维护需求。建议将此文加入书签，随时查阅。