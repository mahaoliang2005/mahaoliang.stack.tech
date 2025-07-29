---
title: "使用 1Password 和 SSH Agent Forwarding 提升远程开发体验"
date: 2025-07-29T21:05:21+08:00
draft: false
tags: [macos, linux, ssh, git]
categories: [tech]
---
作为开发者，我们经常需要通过 SSH 连接到远程 Linux 服务器进行开发。工具如 VS Code 的 Remote-SSH 插件，让我们几乎感觉不到自己是在一台远程机器上工作。但一个常见的痛点随之而来：**SSH 密钥管理**。

我们希望：
1.  使用 SSH 密钥登录服务器，并向 Git 仓库（如 GitHub）推送代码。
2.  对每一次 Git 提交进行签名，以验证提交的来源。
3.  最重要的一点：**不希望将包含私钥的任何文件拷贝到远程服务器上**，以防服务器被入侵导致私钥泄露。

幸运的是，通过 1Password 内置的 SSH Agent 和 SSH Agent Forwarding 技术，我们可以完美地解决这个问题。本文将带你一步步配置，实现安全、无缝的远程开发流程。

## 核心概念简介

在开始之前，我们先简单了解几个关键概念：

*   **SSH Agent (SSH 代理)**：它就像一个临时的密钥管理员。你可以在会话开始时，将解密的私钥（decrypted private key）加载到 Agent 中。之后，任何需要使用该密钥的 SSH 操作都会向 Agent 请求，而无需你反复输入密码。当你关闭终端会话时，Agent 也会随之关闭，密钥被安全地清除。

*   **1Password SSH Agent**：1Password 8 及以上版本内置了一个功能强大的 SSH Agent。它将你的 SSH 私钥安全地存储在 1Password 保管库中，并通过一个安全的套接字文件（socket file）与你的系统交互。这意味着你的私钥永远不会以明文形式存在于磁盘上，所有使用请求都需要经过 1Password 的授权（例如 Touch ID 或主密码）。

*   **SSH Agent Forwarding (SSH 代理转发)**：这是一个非常强大的 SSH 功能。当你从本地电脑 SSH 到远程服务器时，它可以建立一个安全通道，将远程服务器上需要密钥认证的操作请求，“转发”回你的**本地电脑**，交由你本地的 SSH Agent 来处理。这样一来，远程服务器本身完全不需要存储任何私钥。

## 我们的目标流程

1.  在本地电脑上，1Password 管理着我们的 SSH 私钥。
2.  通过 VS Code Remote-SSH 或终端连接到远程服务器，并启用 Agent Forwarding。
3.  在远程服务器上，执行 `git push` 时，认证请求被转发回本地，由 1Password 处理。
4.  在远程服务器上，执行 `git commit` 时，签名请求也被转发回本地，由 1Password 处理和授权。

## 配置流程

### 前提条件

*   你已经安装了 1Password 8 或更高版本的桌面客户端。
*   你的 SSH 密钥已经创建并保存在 1Password 的 `SSH 密钥` 分类中。

### 第一步：配置本地电脑，让 SSH 使用 1Password

首先，我们需要告诉本地的 SSH 客户端，让它把所有密钥相关的请求都交给 1Password 处理。

1.  **在 1Password 中启用 SSH Agent**
    *   打开 1Password 桌面应用。
    *   进入 `设置` -> `开发者`。
    *   勾选 `使用 SSH 代理`。
![ssh agent](https://cdn.mahaoliang.tech/2024/202507292109013.jpg)

2.  **配置本地 SSH 配置文件 (`~/.ssh/config`)**

    参考 1password 的[官方文档](https://developer.1password.com/docs/ssh/get-started/#step-4-configure-your-ssh-or-git-client)，我们有两种方式告诉 SSH 客户端 Agent 在哪里：`IdentityAgent` 指令和 `SSH_AUTH_SOCK` 环境变量。推荐使用 `IdentityAgent`。

    编辑你**本地电脑**上的 `~/.ssh/config` 文件（如果不存在，请创建它）。在文件顶部添加以下内容：

    ```sh
    # 告诉所有 SSH 连接 (*) 都使用 1Password 的 Agent
    Host *
      IdentityAgent "~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
    ```
    
3.  **验证配置**

    在**本地电脑**的终端里运行以下命令：

    ```sh
    ssh-add -l
    ```

    如果配置成功，它会列出你在 1Password 中存储的所有 SSH 密钥的公钥指纹。这证明你的本地 SSH 客户端已经成功与 1Password 对接。

### 第二步：配置连接，启用 Agent Forwarding

现在，我们需要在连接到特定远程服务器时，启用 Agent Forwarding 功能。最佳实践依然是修改 `~/.ssh/config` 文件。

继续编辑你**本地电脑**上的 `~/.ssh/config` 文件，为你的服务器添加一个专有配置块：

```sh
# 给你的远程服务器起一个别名，方便连接
Host my-dev-server
    HostName <your_server_ip_or_domain>
    User mahaoliang
    ForwardAgent yes   # <-- 关键！启用 Agent Forwarding
```

*   `Host my-dev-server`: 这是你连接时使用的快捷别名。
*   `HostName`: 服务器的实际 IP 或域名。
*   `User`: 你在服务器上的用户名。
*   `ForwardAgent yes`: 这就是开启 Agent Forwarding 的开关。

现在，你可以通过 `ssh my-dev-server` 或在 VS Code Remote-SSH 中直接连接到 `my-dev-server`，转发功能会自动启用。

### 第三步：配置远程服务器上的 Git

这是最后一步，也是最关键的一步。我们需要告诉远程服务器上的 Git，如何使用我们转发过来的 SSH Agent 进行提交签名。

1.  **连接并验证转发**

    首先，连接到你的远程服务器：
    ```sh
    ssh my-dev-server
    ```
    连接成功后，在**远程服务器**的终端上，再次运行验证命令：
    ```sh
    ssh-add -l
    ```
    如果 Agent Forwarding 正常工作，这里显示的输出应该和你**本地电脑**的输出**完全一样**！如果提示 "The agent has no identities."，请返回第二步检查配置。

2.  **获取用于签名的公钥**

    Git 需要知道用哪个具体的密钥来签名。我们需要提供完整的公钥字符串作为标识。在**远程服务器**上运行：

    ```sh
    ssh-add -L
    ```

    这个命令会列出 Agent 中所有密钥的完整公钥。复制你想要用来签名的那一行，它看起来像这样：
    `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICxxxxxxxxxxxxxxxxxxxx your-key-comment`

3.  **配置远程服务器的 `.gitconfig`**

    现在，编辑你**远程服务器**上的 `~/.gitconfig` 文件。将你原来的配置更新如下：

    ```ini
    [user]
        email = mahaoliang@gmail.com
        name = mahaoliang
        # 将 signingkey 的值设置为你上一步复制的完整公钥字符串
        signingkey = ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICxxxxxxxxxxxxxxxxxxxx your-key-comment
    
    [gpg]
        # 告诉 Git 使用 ssh 程序进行签名
        format = ssh
    
    [commit]
        # 让所有提交都默认进行签名
        gpgsign = true
    ```

    最重要的改动在于 `signingkey`。我们不再使用一个文件路径，而是直接提供了公钥本身。这让 Git 可以直接向转发过来的 Agent 请求使用这个特定的密钥进行签名。

### 大功告成！来测试一下吧

一切准备就绪！在远程服务器上，进入你的任意一个 git 项目，尝试创建一个新提交：

```sh
git commit --allow-empty -m "Test: Signed commit with 1Password Agent Forwarding"
```

此时，奇妙的事情发生了：你的**本地电脑**上会弹出 1Password 的授权请求，提示你应用正在请求使用你的 SSH 密钥。通过 Touch ID 或输入主密码授权后，远程服务器上的 `git commit` 命令瞬间完成。

![ssh key](https://cdn.mahaoliang.tech/2024/202507292112652.jpg)

最后，检查一下你的提交日志：

```sh
git log --show-signature -1
```

你将会看到类似下面的输出，`Good signature` 明确告诉你，这次提交已经由你的密钥成功签名！

```
commit <commit_hash> (HEAD -> main)
Good "git" signature for mahaoliang@gmail.com with ED25519 key SHA256:GKaU0ZCgehQ73X...
Author: mahaoliang <mahaoliang@gmail.com>
Date:   ...

    Test: Signed commit with 1Password Agent Forwarding
```

### 总结

通过以上配置，我们构建了一个既安全又便捷的远程开发工作流。你的私钥始终安全地躺在本地的 1Password 保管库中，而远程服务器上的所有 Git 操作（认证和签名）都能够无缝、安全地使用它。这不仅提升了安全性，也大大简化了多服务器环境下的密钥管理，让你能更专注于编码本身。
