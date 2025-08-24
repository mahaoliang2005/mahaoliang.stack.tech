---
title: "一份给开源新手的 GitHub 贡献流程指南"
date: 2025-08-24T16:26:32+08:00
draft: false
tags: [opensource, git, github]
categories: [tech]
---
作为一名开发者，参与开源是提升技术、建立个人影响力的有效途径。但很多人，包括曾经的我，在刚接触时都会被 GitHub 的协作流程搞得一头雾水。

比如，什么是 `Fork`？`Upstream` 和 `Origin` 又有什么区别？标准的贡献流程是怎样的？网上虽然有资料，但很少有一篇文章能为新手清晰地讲明白整个过程。

今年我参加了“[开源之夏](https://summer-ospp.ac.cn/)”活动，在为 openEuler 社区的 [splitter](https://gitee.com/openeuler/splitter) 项目提交 [PR](https://gitee.com/openeuler/splitter/pulls/19) 的过程中，才算真正搞清楚完整的流程。因为自己经历过初期的困惑，所以决定把这个过程完整地记录下来，形成一篇能直接上手操作的指南。

这篇文章的目的很明确：就是为那些想参与开源但对流程不熟悉的开发者，提供一份清晰、实用的操作手册。希望这篇总结能帮你扫清障碍，顺利迈出开源贡献的第一步。下面我们正式开始。

## 核心概念解析

在动手操作之前，我们必须先理解几个关键的名词。搞清楚它们之间的关系，是顺利完成后续所有步骤的基础。

### Git 基础概念回顾

首先，你需要对 Git 本身的工作流程有一个基本了解。下面这张图很经典地展示了 Git 的核心区域和操作：

![git](https://cdn.mahaoliang.tech/2024/202508241656520.png)

简单来说，你的日常工作就是：

1.  在 **Workspace (工作区)** 修改代码。
2.  使用 `git add` 将修改内容暂存到 **Index (暂存区)**。
3.  使用 `git commit` 将暂存区的内容提交到本地 **Repository (仓库)**，形成一个版本记录。
4.  使用 `git push` 将本地仓库的提交推送到 **Remote (远程仓库)**，比如 GitHub。
5.  使用 `git pull` 或 `git fetch` 从远程仓库拉取更新。

对于这些基础命令的详细用法，阮一峰老师[《常用 Git 命令清单》](https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)已经总结得非常全面，是很好的速查手册，本文不再赘述。

### GitHub 开源协作的概念

上面提到的模型只涉及一个本地仓库和一个远程仓库。但在开源协作中，通常会涉及**三个仓库**。下面这张图清晰地展示了它们的关系，这也是本节的重点：

![github](https://cdn.mahaoliang.tech/2024/202508241704135.png)

让我们结合这张图，来理解几个最重要的概念：

*   **Upstream (上游仓库)**
    *   **是什么**：图中左上角的 "GitHub - Original"，也就是你想要贡献代码的那个**原始开源项目仓库**。
    *   **你的权限**：你通常没有直接往这里 `push` 代码的权限。

*   **Fork (你的个人复刻)**
    *   **是什么**：图中右上角的 "GitHub - Fork"。这是你通过在原始项目页面上点击 "Fork" 按钮，在**你自己的 GitHub 账号下创建的一个完整的项目副本**。
    *   **你的权限**：因为这个仓库在你的名下，所以你拥有全部的读写权限，可以随意 `push` 代码。这是你为开源项目贡献代码的“大本营”和“实验区”。

*   **origin (你的远程仓库)**
    *   **是什么**：简单来说，`origin` 是 Git 为你克隆的那个远程仓库地址起的一个**默认别名**。
    *   **如何创建**：当你从 GitHub 克隆你的 Fork 仓库时，通过执行 `git clone [URL of YOUR FORK]` 这个命令，Git 会在你的本地仓库中**自动创建**这个名为 `origin` 的别名。你不需要任何手动设置。
    *   **指向哪里**：在我们的协作流程中，`origin` 这个别名指向的就是**你自己的 Fork 仓库** (也就是图右上角的 "GitHub - Fork")。
    *   **作用**：它是你 `push` 本地代码时的默认目标。当你执行 `git push` 时，你的代码变更就是通过 `origin` 这个别名，被推送到你自己的 Fork 仓库中。

*   **upstream vs. origin 的关系**
    *   **`origin`** 指向你自己的 Fork，是你**推送 (push) 代码**的地方。
    *   **`upstream`** 指向原始项目仓库，是你 **同步最新代码 (fetch)** 的地方。
    *   **关键点**：`origin` 是克隆时自动生成的，而 `upstream` 是需要我们**手动添加**的。这一步至关重要，它建立了你的本地仓库和原始项目之间的连接，让你能够随时获取项目的最新进展。

*   **Pull Request (PR - 合并请求)**
    *   **是什么**：当你觉得在自己 Fork 里的代码已经准备好，可以贡献给原始项目时，你就可以在 GitHub 上发起一个 Pull Request。
    *   **作用**：这是一个正式的请求，请求 `Upstream` (原始项目) 的维护者，把你 Fork 里的代码变更拉取 (Pull) 到他们的仓库中。这也是代码审查 (Code Review)、讨论和最终合并的地方。

**总结一下**，整个流程的数据流是这样的：

1.  **初始化：从 `origin` 到本地**

    你首先 `clone` 的是你自己的 Fork 仓库 (`origin`)，把代码从你的 GitHub 仓库复制到本地电脑。这是你建立本地工作环境的第一步。

2.  **开发与同步：`upstream` -> 本地 -> `origin`**

    这个阶段是循环往复的。
    *   **获取更新**：你需要定期从 `upstream` (原始项目) `fetch` 或 `rebase` 最新的代码到本地，确保你的工作是基于最新版本。
    *   **推送你的贡献**：在本地开发完成后，你把代码 `push` 到 `origin` (你自己的 Fork)。你的代码变更只会上传到你自己的远程仓库里。

3.  **提交贡献：从 `origin` 到 `upstream`**

    最后，你通过在 GitHub 上创建一个 Pull Request，请求 `upstream` (原始项目) 的维护者，来审核并合并你 `origin` (你的 Fork) 里的代码。

这个流程确保了你的所有修改都在自己的“地盘”上进行，既不会影响到原始项目，又能随时与原始项目保持同步，并最终通过 PR 的方式发起贡献。

理解了这三个仓库和它们之间的关系，我们就已经扫清了最大的障碍。接下来，我们将进入实战环节。

## 首次贡献实战演练

理论知识已经储备完毕，现在让我们卷起袖子，完整地走一遍实际的贡献流程。我们会模拟为一个名为 `project-name` 的开源项目贡献代码。

### Step 1: Fork - 拥有你自己的副本

首先，你需要进入 `project-name` 的 GitHub 主页。在页面的右上角，你会看到一个 "Fork" 按钮。点击它，GitHub 就会在你的个人账号下创建一个该项目的完整副本。

![fork](https://cdn.mahaoliang.tech/2024/202508241744175.webp)

完成后，你的 GitHub 主页上就会出现一个 `your-username/project-name` 的仓库。这就是你的个人复刻 (Fork)。

### Step 2: Clone - 将代码克隆到本地

现在，你需要把你 Fork 的仓库克隆到你的电脑上进行开发。

进入你刚刚创建的 `your-username/project-name` 仓库页面，点击绿色的 "Code" 按钮，复制 HTTPS 或 SSH 链接。

![clone](https://cdn.mahaoliang.tech/2024/202508241745385.webp)

然后，在你的电脑终端中执行以下命令：

```bash
# 确保将 YOUR-USERNAME 替换为你的 GitHub 用户名
git clone https://github.com/YOUR-USERNAME/project-name.git
```

**注意**：这里克隆的是**你自己的 Fork 仓库地址**，而不是原始项目的地址。这是非常关键的一步。

### Step 3: 配置 upstream，建立与上游的连接

为了能够随时获取原始项目的更新，我们需要在本地配置一个指向上游仓库 (`upstream`) 的远程地址。

首先，进入项目目录：

```bash
cd project-name
```

然后，添加 `upstream`：

```bash
# 将 URL 替换为原始开源项目的 URL
git remote add upstream https://github.com/original-owner/project-name.git
```

我们可以用 `git remote -v` 命令来检查是否配置成功。你应该能看到类似下面的输出：

```bash
origin    https://github.com/YOUR-USERNAME/project-name.git (fetch)
origin    https://github.com/YOUR-USERNAME/project-name.git (push)
upstream  https://github.com/original-owner/project-name.git (fetch)
upstream  https://github.com/original-owner/project-name.git (push)
```
看到 `origin` 和 `upstream` 同时存在，就说明配置成功了。

### Step 4: 创建特性分支

在开始写代码之前，一个最佳实践是为你的新功能或修复创建一个独立的分支。这能确保你的 `master` 分支保持干净，并与上游项目同步。

首先，确保你的本地 `master` 分支是最新的：

```bash
git fetch upstream
git checkout master
git rebase upstream/master
```

然后，从最新的 `master` 分支上创建一个新分支：

```bash
# 把 my-feature-branch 换成一个有意义的名字，比如 fix-login-bug
git checkout -b my-feature-branch
```
现在，你就可以在这个新分支上安全地进行开发了。

### Step 5: 开发与提交

在这个分支上，你可以自由地修改代码、添加新文件、修复 Bug。完成一个阶段性的工作后，就进行一次 `commit`。

```bash
# 1. 添加你的修改到暂存区
git add .

# 2. 提交你的修改，并写下清晰的提交信息
git commit -m "feat: Implement user profile page"
```

### Step 6: 保持同步，与上游代码对齐 (可选但重要)

如果你的开发周期比较长，在你开发期间，`upstream` 可能已经合并了其他人的代码。为了避免提交 PR 时产生冲突，建议在推送前，先同步一次上游的最新代码。

```bash
git fetch upstream
git rebase upstream/master
```

`rebase` 可以让你的提交历史保持一条直线，看起来更整洁。如果遇到冲突，你需要先解决冲突，然后继续 `rebase` 过程。

### Step 7: 推送，将变更推送到你的 Fork

当你在本地完成开发和提交后，就可以把你的特性分支推送到你自己的 Fork 仓库 (`origin`) 了。

```bash
git push --set-upstream origin my-feature-branch
```

`--set-upstream` 参数只需要在第一次推送这个分支时使用，它会告诉 Git 你的本地分支 `my-feature-branch` 对应远程仓库 `origin` 的同名分支。

### Step 8: 创建 Pull Request

推送成功后，现在回到你在 GitHub 上的 Fork 仓库页面。通常，GitHub 会自动检测到你推送了新的分支，并显示一个黄色的提示条，让你方便地创建 Pull Request。

![PR](https://cdn.mahaoliang.tech/2024/202508241756895.webp)

点击 "Compare & Pull Request" 按钮，你会进入 PR 创建页面。在这里，你需要：
1.  **填写一个清晰的标题**：简明扼要地说明这个 PR 的作用。
2.  **撰写详细的描述**：解释你为什么要进行这些修改，解决了什么问题，以及你是如何实现的。如果项目有 PR 模板，请务必按照模板填写。

确认无误后，点击 "Create pull request"。恭喜你，你的第一个 PR 已经成功提交了！

### Step 9: 响应代码审查 (Code Review)

提交 PR 只是开始，接下来你需要和项目维护者进行互动。

1.  **维护者留下评论**：项目维护者会审查你的代码，并在可能有问题的地方留下评论 (Comment)。你会在 GitHub PR 页面和邮件中收到通知。

![comment](https://cdn.mahaoliang.tech/2024/202508241800186.png)

2.  **回复与讨论**：对于每一条评论，你都应该进行回复。如果同意修改，可以简单回复“好的，马上修改”；如果不理解或有不同意见，可以在评论区进行礼貌的讨论。

3.  **修改代码并再次提交**：
    *   回到你的本地电脑，确保你还在之前的 `my-feature-branch` 分支上。
    *   根据讨论结果，直接修改代码。
    *   修改完成后，创建一个新的 `commit`：

        ```bash
        git add .
        git commit -m "fix: Address review comments from maintainer"
        ```

    *   **重要**：修改完成后，直接将新的提交推送到你的分支：

        ```bash
        git push origin my-feature-branch
        ```

    *   你**不需要**重新创建一个 PR。这次 `push` 会自动更新你之前提交的那个 PR。

4.  **解决对话 (Resolve Conversation)**：

    当你认为你已经解决了某一条评论中的问题后，回到 PR 页面，找到对应的评论，点击 "**Resolve conversation**" 按钮。这是一种礼貌的表示，告诉维护者这个问题你已经处理完毕，方便他们再次审查。

这个修改 -> 推送 -> 解决对话的循环可能会进行多次，直到所有问题都得到解决。

### Step 10: 合并

当你的代码通过了所有审查，维护者就会点击 "Merge pull request" 按钮，将你的代码合并到主项目中。

![merged](https://cdn.mahaoliang.tech/2024/202508241810964.png)

至此，你的代码就正式成为了开源项目的一部分！你可以在项目的贡献者名单中看到自己的名字，这是对你辛勤付出的最好回报。

## 保持更新，如何为下一次贡献做准备

恭喜你！在完成了一次 PR 合并后，你已经是一位正式的开源贡献者了。但工作还没结束。为了能方便地进行下一次贡献，你需要学会如何让你的 Fork 仓库与原始项目保持同步。

### 为什么需要同步你的 Fork

在你完成一次贡献后，原始项目（`upstream`）的 `master` 分支因为合并了你的 PR 和其他人的代码，已经向前更新了。而你自己的 Fork 和本地仓库的 `master` 分支，还停留在你开始工作时的旧位置。

如果你不进行同步，直接从一个过时的 `master` 分支上创建新分支进行开发，那么在提交下一次 PR 时，几乎必然会遇到大量的代码冲突，给自己和项目维护者都带来不必要的麻烦。

因此，**在开始下一次贡献前，保持你的 `master` 分支与 `upstream` 完全同步，是一个至关重要的好习惯。**

### 两步完成同步

整个同步过程非常简单，可以分为两步：先更新本地，再更新你的远程 Fork。

#### 第一步：更新你的本地仓库

1.  **切换到主分支**

    首先，确保你回到了 `master` 分支。所有同步操作都应该在 `master` 分支上进行。

    ```bash
    git checkout master
    ```

2.  **从上游拉取最新变更**

    这个命令会从原始项目（`upstream`）下载最新的代码历史，但**不会**自动修改你本地的任何文件。

    ```bash
    git fetch upstream
    ```

3.  **将本地 master 同步到上游 master**

    这步是关键。它会以 `upstream/master` 的最新版本为基础，来更新你本地的 `master` 分支。执行后，你的本地 `master` 分支就和原始项目的 `master` 分支完全一致了。

    ```bash
    git rebase upstream/master
    ```

现在，你的本地 `master` 分支已经是最新版本了。

#### 第二步：更新你在 GitHub 上的 Fork

你的本地 `master` 是最新的了，但你 GitHub 上的 Fork (`origin`) 还停留在旧版本。我们需要把本地的更新推送到 `origin`。

```bash
git push origin master
```

执行这个命令后，你 GitHub 上的 Fork 仓库的 `master` 分支也会被更新到最新状态。

### 小结

现在，你的**本地仓库**和 **GitHub 上的 Fork** 都和上游项目完全同步了。

你可以安心地从这个干净、最新的 `master` 分支上，通过 `git checkout -b new-feature` 创建新的特性分支，开始你的下一次贡献了。

记住这个流程：**每次开始新工作前，先同步 `master` 分支**。这将让你的开源贡献之路更加顺畅。

## 常用命令速查清单 (Cheat Sheet)

在熟悉了整个流程后，你不需要每次都回头阅读长篇的文字。这份速查清单总结了在不同阶段最核心的命令，你可以把它当作日常贡献时的“小抄”。

### 首次设置 (为一个新项目贡献时，仅需一次)

```bash
# 1. 克隆你自己的 Fork 仓库到本地
git clone [URL of YOUR FORK]

# 进入项目目录
cd [project-name]

# 2. 添加原始项目仓库为 upstream
git remote add upstream [URL of ORIGINAL REPO]

# 3. 验证远程仓库设置是否成功
git remote -v
```

### 一个完整的开发周期

```bash
# === 准备阶段 ===

# 1. 切换到主分支
git checkout master

# 2. 与上游 master 分支保持同步
git fetch upstream
git rebase upstream/master

# 3. 从最新的 master 分支创建你的特性分支
git checkout -b [my-new-feature-branch]


# === 开发阶段 ===

# ... 在这里进行代码的修改、添加、删除 ...

# 1. 添加变更到暂存区
git add .

# 2. 提交变更
git commit -m "feat: Add a new amazing feature"


# === 提交阶段 ===

# 1. (可选但推荐) 在推送前，再次与上游同步，避免冲突
git fetch upstream
git rebase upstream/master

# 2. 将你的分支推送到你自己的 Fork 仓库 (origin)
# 第一次推送时使用 --set-upstream
git push --set-upstream origin [my-new-feature-branch]
```

推送完成后，去 GitHub 页面创建 Pull Request。

### 响应代码审查 (Review 后修改代码)

```bash
# 确保你还在你的特性分支上
# ... 直接修改代码 ...

# 1. 提交你的修改
git add .
git commit -m "fix: Address review comments"

# 2. 再次推送到你的分支，PR会自动更新
git push origin [my-new-feature-branch]
```

### 贡献完成后，保持 Fork 更新 (为下一次贡献做准备)

```bash
# 1. 切换到主分支
git checkout master

# 2. 拉取上游的最新代码
git fetch upstream

# 3. 将本地 master 更新到最新
git rebase upstream/master

# 4. 将这个最新的 master 分支推送到你自己的 Fork，使其在 GitHub 上也保持最新
git push origin master
```

## 结语

到这里，我们已经完整地走完了在 GitHub 上参与一个开源项目的全过程：从理解 Fork、Upstream 这些核心概念，到一步步创建分支、提交代码、发起 Pull Request，再到如何与维护者互动以及如何为下一次贡献保持仓库同步。

其实，整个流程可以被总结为一条清晰的路径：

**Fork -> Clone -> Branch -> Develop -> Push -> PR**

开源贡献的流程并不复杂，最重要的，是勇敢地迈出第一步。

希望这篇文章能成为你开启开源之旅的得力助手。现在，就去寻找一个你感兴趣的项目，提交你的第一个 PR 吧！