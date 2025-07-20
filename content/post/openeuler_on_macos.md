---
title: "macOS 上 VMWare Fusion 无法安装 OpenEuler 的问题解决"
date: 2025-07-07T20:26:30+08:00
draft: false
tags: [linux, macos, openEuler]
categories: [tech]
---

今年我非常幸运地入选了“开源之夏”活动，参与的项目是[面向 openEuler distroless 镜像的 SDF 自动生成工具开发](https://summer-ospp.ac.cn/org/prodetail/25b970448)。由于该项目是在 openEuler 上进行开发的，因此，搭建一个稳定可靠的 openEuler 开发环境便成了我的首要任务。

我的主力设备是一台 M1 芯片的 MacBook Pro，由于去年 [VMware Fusion 已经对个人用户免费](https://blogs.vmware.com/cloud-foundation/2024/11/11/vmware-fusion-and-workstation-are-now-free-for-all-users/)，所以自然就选择在 VMware Fusion 上安装 openEuler。

从 openEuler 官方镜像站下载 aarch64 架构最新的 openEuler 24.03 LTS SP1 镜像，使用 VMware Fusion 创建虚拟机，挂载镜像，启动后在 GRUB 界面选择“Install openEuler”，没想到就遇到了问题，安装过程卡住不动了：

![installation frozen](https://cdn.mahaoliang.tech/2024/202507172109472.png)

习惯性的截图问 [Google AI studio](https://aistudio.google.com/)，它列举了多个可能的原因，逐一排除确认，发现其中一个分析很有启发性：

> 这个问题可能是由于内核在初始化虚拟显卡时卡住了

于是尝试在 openEuler 的安装引导菜单，按 e 键进入编辑模式：

![install](https://cdn.mahaoliang.tech/2024/202507172140190.png)

继续截图让 Google AI studio 分析：

> 您看到的这个界面是 GRUB 引导加载器的编辑模式。它允许您在启动 Linux 内核之前，临时修改传递给内核的参数。

>    *   `video=efifb:off video=VGA-1:640x480...`: **这是一个重要线索**！这行参数试图禁用 EFI 帧缓冲（`efifb:off`）并强制使用一个非常基础的 640x480 分辨率的 VGA 模式。这说明 openEuler 镜像为了在各种（尤其是老旧或奇怪的）硬件上都能启动而加入的“安全模式”参数。然而，这个配置与您的 VMware Fusion 环境冲突，导致了卡死。

> **请按照以下步骤操作：**

> 1.  在 `linux` 那一行，**手动删除**掉现有的两个 `video=...` 参数，即删除 `video=efifb:off` 和 `video=VGA-1:640x480-32@60me`。
> 2.  按 `Ctrl+X` 或 `F10` 启动。

按照 AI 的指示操作，奇迹发生了，系统顺利越过了卡死点，成功进入了图形化的安装界面！

Gemini 2.5 Pro 太厉害了，遇事不会问 Gemini，准没错！
