---
title: "一步步教你在 VMware Fusion 中实现与 Linux 虚拟机的文件共享"
date: 2025-07-09T11:41:57+08:00
draft: false
tags: [linux, macos, vmware]
categories: [tech]
---
对于许多在 macOS 上工作的开发者来说，VMware Fusion 就像一把瑞士军刀，让我们能随时拥有一个纯净、独立的 Linux 环境。无论是进行 Linux 系统编程、用 Python 做 Linux 系统工具开发，还是部署和测试应用程序，虚拟机都为我们提供了一个完美的沙盒。

然而，当你将所有的代码、项目文件和配置文件都存放在 Linux 虚拟机中，万一哪天虚拟机意外崩溃、无法启动，所有的心血都可能付诸东流。

好消息是，VMware Fusion 提供了一个极其强大的功能，**“共享文件夹”（Shared Folders）**，通过简单的配置，我们可以将 macOS 上的任意一个文件夹，直接“挂载”到 Linux 虚拟机的系统中，让它看起来就像是 Linux 自己的一个目录。

这意味着，你可以在 macOS 或 Linux 上修改代码，因为它们都是同一份，同时你也可以使用 Time Machine 或其他云盘来实现代码的备份，即方便又安全。

本文将作为你的向导，手把手带你完成从 VMware Fusion 的配置，到 Linux 系统配置的全部过程。让我们开始吧！

## 准备工作，安装 VMware Tools

在开始配置共享之前，我们需要确保环境准备就绪，已经在 VMware Fusion 中安装好一个 Linux 发行版。

接下来需要在 Linux 虚拟中安装 **VMware Tools**。

你可以把 **VMware Tools** 理解为是连接 macOS 宿主机和 Linux 虚拟机的“桥梁”或“驱动程序”。像虚拟机屏幕分辨率自适应、鼠标无缝切换、剪贴板共享，以及我们本次的目标**文件夹共享**，都完全依赖于它。

对于大多数的 Linux 发行版，官方和社区共同维护了一个开源实现，叫做 [open-vm-tools](https://github.com/vmware/open-vm-tools)，它已经预置在大多数 Linux 发行版的软件源中，安装起来非常方便。

`open-vm-tools` 主要由以下几个软件包组成：

*   `open-vm-tools`: 这是核心包，提供了最基础的功能，如虚拟机时钟同步、与宿主机的电源操作（正常关机）、心跳检测，以及最重要的，它包含了实现文件夹共享所必需的组件。

*   `open-vm-tools-desktop`: 它在核心包的基础上，增加了改善图形化交互体验的功能，例如剪贴板复制粘贴、窗口大小自适应等。

*   `open-vm-tools-devel` 和 `open-vm-tools-debuginfo`: 这两个包分别用于二次开发和调试，普通用户完全不需要关心。

了解了这些，我们的目标就非常明确了。在虚拟机的命令行终端执行以下命令：

```bash
# Ubuntu
sudo apt update
sudo apt install open-vm-tools

# RHEL
sudo yum install open-vm-tools
```

安装完成后，重启虚拟机，确保所有服务都能正常加载。下一步，我们就去 VMware Fusion 中开启文件共享功能。

## 在 VMware Fusion 中配置共享

先将 Linux 虚拟机关机，然后进入虚拟机的设置面板，点击“共享”图标，勾选 **“启用共享文件夹”** 这个复选框，接着，点击下方的 `+` 号按钮，准备添加一个具体的共享目录。

![启用共享文件夹](https://cdn.mahaoliang.tech/2024/202507191233366.png)

确保“启用”是勾选状态，并且权限设置为“读与写”，这样你才能在 Linux 中创建和修改文件。

关闭配置，启动虚拟机，接下来需要在 Linux 中完成共享目录的挂载，就可以访问共享文件夹了。

## 在 Linux 虚拟机中访问和挂载

在最新版的 `open-vm-tools` 的支持下，VMware 的共享文件夹通常会被自动挂载到一个系统级的公共目录：`/mnt/hgfs` (Host-Guest File System)。

我们可以先验证一下。打开终端，运行：

```bash
ls /mnt/hgfs
```

如果能看到你在上一步设置的共享名（如 linux），那么恭喜你，已经成功了！

### 手工挂载

有时 `/mnt/hgfs` 目录是空的，这可能是因为权限或 FUSE 服务问题。解决方法是使用 `vmhgfs-fuse` 命令手动挂载。

```bash
# .host:/ 是一个特殊地址，代表所有已启用的共享
# /mnt/hgfs 是挂载点
# -o allow_other  允许其他用户(包括你自己)访问，非常重要！
# -o uid=$(id -u)  将文件所有者设置为当前登录的用户
# -o gid=$(id -g)  将文件所属组设置为当前登录的用户
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other -o uid=$(id -u) -o gid=$(id -g)
```

现在 `/mnt/hgfs` 目录下就可以看到共享的文件了。

不过，`/mnt/hgfs/linux` 这个路径太深，不方便日常使用，我们想将共享目录挂载在用户主目录下的 `~/works` 目录中。

首先在用户主目录下创建 `works` 目录作为挂载的目标。

```bash
mkdir ~/works
```

然后同样使用 `vmhgfs-fuse` 工具来执行挂载。

```bash
sudo vmhgfs-fuse .host:/ ~/works -o allow_other -o uid=$(id -u) -o gid=$(id -g)
```

现在，来验证一下命令效果：

```bash
ls ~/works
```

你应该能看到 `linux` 目录。再进一步查看：

```bash
ls ~/works/linux
```

此刻，你看到的就是 macOS 宿主机上那个共享文件夹里的所有内容了！

然而手动挂载是临时的，一旦你重启虚拟机，挂载就会失效。要实现一劳永逸，请看下一步。

### 实现开机自动挂载

为了避免每次重启都要手动敲一遍命令，我们需要将挂载信息写入 `/etc/fstab` 文件中。

在 `fstab` 中，我们不能使用 `$(id -u)` 这样的命令。需要把用户 ID 和组 ID 的**具体数字**写进去。运行以下命令查看：

```bash
id
# 你会看到类似 uid=1000(ubuntu) gid=1000(ubuntu) ... 的输出
```

通常，第一个创建的用户的 UID 和 GID 都是 `1000`。请记下你自己的这两个数字。

使用你熟悉的编辑器打开 `/etc/fstab`。

```bash
sudo vi /etc/fstab
```

在文件的末尾，添加下面这一行。**请注意，你需要将 `/home/ubuntu/works` 替换成你的实际路径，并将 `uid=1000,gid=1000` 替换成你自己的 ID。**

```bash
.host:/ /home/ubuntu/works fuse.vmhgfs-fuse defaults,allow_other,uid=1000,gid=1000 0 0
```

以后每次启动 Linux 虚拟机，VMware 的共享文件夹都会自动出现在 `~/works` 目录下，方便随时访问。

大功告成！