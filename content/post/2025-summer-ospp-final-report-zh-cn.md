---
title: "开源之夏结项报告"
date: 2025-09-04T00:00:00+08:00
draft: false
tags: [linux, openEuler, docker]
categories: [tech]
---
今年暑假最大的收获就是拿到了驾照，然后就是参加了[开源之夏 (Summer OSPP)](https://summer-ospp.ac.cn/) 。经过两个月的开发，今天终于可以为 [面向 openEuler distroless 镜像的 SDF 自动生成工具开发](https://summer-ospp.ac.cn/org/prodetail/25b970448) 项目提交了结项报告了。一个合并了 7 个 PR，完全超出了我的预期。

下面是我的结项报告。

---

# 结项报告

- **学生姓名：** 马浩量
- **项目编号：** 25b970448

## 项目信息

*   **项目名称**：面向 openEuler distroless 镜像的 SDF 自动生成工具开发

*   **方案描述**：
    本项目旨在为 openEuler 的 [splitter](https://gitee.com/openeuler/splitter) 工具开发一个全新的 `gen` 命令，以实现 Slice Definition File (SDF) 的自动化生成。当前所有用于构建 Distroless 镜像的 SDF 文件均需手工编写，效率低而且容易出错。
    
    本方案通过设计一套自动化流水线，为任意 RPM 包生成高质量的 SDF 草稿。其核心流程包括：在隔离的 Docker 环境中动态构建一个依赖完备的分析环境，然后对目标 RPM 包进行**智能文件分类**和**精确依赖分析**，最终产出结构清晰、内容准确的 SDF YAML 文件。此工具旨在将社区开发者从繁琐的重复性工作中解放出来，加速 openEuler Distroless 生态的建设。
    
*   **时间规划**：
    
    *   **第一阶段 (7 月 1 日 - 7 月 15 日 )**: **熟悉与框架搭建**，研究 `splitter` 和 `slice-releases` 现有机制，分析手工 SDF 的设计模式，完成 `gen` 命令的基础框架。
    *   **第二阶段 (7 月 16 日 - 7 月 31 日)**: **核心功能实现**。实现基于 `file` 命令的精确文件分类器，基于 `readelf` 的依赖分析器。完成路径压缩、内部依赖注入等 SDF 优化功能。
    *   **第三阶段 (8 月)**: 响应导师建议，引入 Docker 隔离环境，编写 `Dockerfile` 和 `splitter-docker.sh` 运行脚本。向 openEuler 官方镜像仓库贡献 `splitter` 镜像。
    *   **第四阶段 (8 月底)**: 完善 `splitter` 项目的 `README.md`，更新 openEuler Distroless 官方文档，撰写技术博客与结项报告。

## 项目总结

### 已完成工作

对照项目申请书的方案，我完成了预定任务，主要工作成果如下：

1.  **新增 `splitter gen` 命令**:
    *   成功为 `splitter` 工具增加了 `gen` 子命令，提供了从无到有的 SDF 自动化生成能力。
    *   **对应 PR**: [!19](https://gitee.com/openeuler/splitter/pulls/19) (已合并)
2.  **实现文件分类器**:
    *   基于对现有 SDF 的分析，建立了规则引擎，可识别 `_bins`, `_libs`, `_config`, `_copyright` 等核心 slice。
    *   引入 `file -L` 命令进行文件类型校验，确保只有真正的 ELF 文件才会被归入 `_bins` 或 `_libs`，能正确处理脚本和符号链接。
3.  **实现依赖分析器**:
    *   采用“**解析 -> 定位 -> 溯源**”的三步策略，通过 `readelf`, `ldconfig`, `rpm -qf` 工具链，自动化分析 ELF 文件的外部依赖。
    *   实现了内部依赖（如 `_bins` 对 `_config`）的自动注入，使生成的 SDF 更符合设计惯例。
4.  **SDF 写入器**:
    *   实现路径压缩算法，能将版本化的库文件路径（如 `libfoo.so.1`, `libfoo.so.1.2.3`）自动合并为 `libfoo.so.1*`，SDF 产出质量接近手工水平。
5.  **引入 Docker 沙箱化分析环境**:
    *   响应导师建议，实现了**基于官方 `openeuler/splitter` 镜像的分析流程**，解决了对宿主环境的依赖和污染问题。
    *   **贡献了官方 `splitter` 镜像**: 编写了 `Dockerfile` ，将 `splitter` 工具镜像贡献至 [openEuler 官方容器镜像仓](https://gitee.com/openeuler/openeuler-docker-images)。
    *   **对应 PR**: [!866](https://gitee.com/openeuler/openeuler-docker-images/pulls/866) (已合并)， [!1023](https://gitee.com/openeuler/openeuler-docker-images/pulls/1023) (已合并)
6.  **提供入口脚本 (`splitter-docker.sh`)**:
    *   编写了一个用户友好的 `splitter-docker.sh` 脚本，封装了所有 Docker 操作，实现了`cut`和`gen`命令的“一键运行”。
    *   **对应 PR**: [!20](https://gitee.com/openeuler/splitter/pulls/20) (已合并) 
7.  **完善的文档**:
    *   更新了 `splitter` 项目的 `README.md`，介绍了 `gen` 命令的原理和使用方法。
        *   **对应 PR**: [!21](https://gitee.com/openeuler/splitter/pulls/21) (已合并) ，[!23](https://gitee.com/openeuler/splitter/pulls/23)(已合并) 
    *   更新了 openEuler Distroless 镜像官方文档，将 SDF 的获取流程，更新为使用本工具的自动化流程。
        *   **对应 PR**: [!1061](https://gitee.com/openeuler/openeuler-docker-images/pulls/1061) (已合并) 

### 遇到的问题及解决方案

在本次项目中，遇到的主要挑战及解决方案如下：

1.  **分析过程对环境的依赖和污染**
    
    我的初始方案为了解决依赖问题，需要在运行工具的宿主系统上，通过`dnf install`安装待分析的软件包及其依赖。这个过程虽然能让分析跑通，但它会直接修改用户的系统环境，存在污染系统的风险。

    **解决方案**：在导师提出“**使用 chroot 或容器技术隔离环境**”的建议后，我最终选择了更加标准的 Docker 方案。基于官方的应用镜像，为每次分析启动一个干净的容器。在容器内部执行`dnf install`，安装待分析的包，在隔离环境中执行 SDF 的生成。分析结束后，直接销毁容器。
    
2.  **如何安全、准确地分析二进制依赖？**
    
    最初考虑使用`ldd`，但发现它存在执行代码的安全风险。

    **解决方案**：经过研究，我选择了`readelf -d`作为核心分析工具。它作为静态分析器，只关注我们需要的直接依赖。
    
3.  **如何处理脚本、符号链接等“伪可执行文件”**
    
    `readelf`无法处理非 ELF 文件，导致分析`bash`等包含大量脚本或内建命令的包时出现大量错误。

    **解决方案**：引入了`file -L`命令作为“预检”步骤。在分类时，先用`file -L` 鉴定文件类型，只有确认为 ELF 文件后，才将其归入`_bins`或`_libs`，后面再并交由`readelf`分析。

### 测试用例

1.  **单元测试**：为`writer.py`中的路径压缩算法编写了单元测试，覆盖了空集合、单文件、多版本组、混合文件等多种边界情况，确保了 SDF 产出的格式质量。
2.  **功能测试**：使用`splitter-docker.sh`脚本，对多种类型的软件包进行了端到端的 SDF 生成测试，并与手工编写的 SDF 进行比对。

### 后续工作安排

虽然核心功能已经完成，但仍然有需要优化和完善的地方：

1. **脚本依赖分析**：探索对 Python `import`或 Shell 脚本中的命令调用进行静态分析，发现非二进制的依赖关系。

2. **功能性切分探索**：研究为`coreutils`这类复杂工具集提供“功能分组元数据”的可能性，以支持更深层次的自动化切分。
3. **批量生成与验证**：使用本工具对`slice-releases`仓库中尚未覆盖的软件包进行 SDF 的批量生成，进一步扩充 openEuler Distroless 的软件包生态。

---

楼下的星巴克，经常去那儿敲代码，比在家里有氛围，这个项目大部份在那儿完成的。拍张照纪念一下。

![starbucks](https://cdn.mahaoliang.tech/2024/202509041739351.jpg)