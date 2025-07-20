---
title: "Python 虚拟环境管理工具选择建议"
date: 2025-07-20T10:37:55+08:00
draft: false
tags: [python]
categories: [tech]
---
前面我写文章分别介绍了 Python 的两个虚拟环境管理工具 [venv](https://mahaoliang.tech/p/python-%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86venv-%E5%AE%9E%E7%94%A8%E6%8C%87%E5%8D%97/) 和 [Conda](https://mahaoliang.tech/p/%E4%B8%80%E6%96%87%E5%BD%BB%E5%BA%95%E6%90%9E%E6%87%82-python-%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86%E7%A5%9E%E5%99%A8-conda/) ，并进行过适用场景对比。但具体应该如何选择呢，我想分享我个人的选择策略，希望能给你提供一个更具体的参考。

## macOS

在 **MacBook Pro** 上，我一般进行纯 Python 开发，倾向于使用 `pyenv` + `venv` 的组合。`pyenv` 负责管理和切换全局的 Python 版本，`venv` 则为每个项目创建极致轻量的虚拟环境。

这套组合非常优雅，工具链清晰解耦，完全符合这类项目的需求。

## Windows

Windows 上的情况要复杂一些。

在我的 **Windows 笔记本** 上，由于配备了 NVIDIA 显卡，可以用于 AI 和机器学习开发，所以会安装 `miniconda`，但一定要注意：

> **不要“Add Miniconda3 to my PATH environment variable”**，

> 也**不要“Register Miniconda3 as my default Python”**，

> 避免干扰系统。如果想使用 `Conda`，通过“开始菜单”找到并打开 **Anaconda Prompt (Miniconda3)** 来使用。

至于使用哪种虚拟环境管理工具，需要根据场景选择。

### 纯 Python 开发

纯 Python 开发，仍然使用 `venv` 创建虚拟环境。

### AI 开发 

AI 开发，需要依赖复杂的 [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads) 和 [cuDNN](https://developer.nvidia.com/rdp/cudnn-archive)。按道理应该使用 `Conda`，可以一键在虚拟环境中安装 CUDA Toolkit 和 cuDNN 本地依赖。

```bash
conda install pytorch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 pytorch-cuda=12.4 -c pytorch -c nvidia
```
但最新的 pytorch 安装向导 [Start Locally](https://pytorch.org/get-started/locally/)，已经取消了 `Conda` 选项，只能在 [Previous PyTorch Versions](https://pytorch.org/get-started/previous-versions/) 中找到 `Conda` 的安装命令。

PyTorch 官方在 2024 年 10 月 22 日通过 GitHub [Issue](https://github.com/pytorch/pytorch/issues/138506) 正式宣布了这一重要决定：

> "2.5 will be the last release of PyTorch that will be published to the pytorch channel on Anaconda."

至于原因，官方声明中提到，将维护资源集中在用户最常用的平台上，可以提供更好的支持和更优质的用户体验。

那么如果想使用最新版的 pytorch，就不能使用 Conda 了。还是老老实实[参考文档](https://www.gpu-mart.com/blog/install-cudnn-on-windows)，**在全局环境中安装并设置 CUDA Toolkit 和 cuDNN，然后使用 `venv` 创建虚拟环境，拷贝执行 [Start Locally](https://pytorch.org/get-started/locally/) 提供命令，在虚拟环境中安装 PyTorch。**

### 多版本需求

如果你的项目依赖特定的 Python 版本，那么建议使用 Conda，为项目创建指定 Python 版本的虚拟环境。

### 复杂的本地依赖

Conda 能够轻松处理那些依赖复杂底层库，它不仅仅管理 Python 包，它管理的是一个完整的、包含所有底层二进制依赖的软件栈。对于依赖复杂非 Python 组件的项目，Conda 很适合。

