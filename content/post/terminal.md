---
title: "打造一个漂亮的命令行终端"
date: 2022-07-23T15:26:12+08:00
draft: false
tags: [macos]
categories: [tech]
---

## 基本设置

打开终端的**偏好设置**，点击**描述文件**tab，将 **Pro** 设置为默认描述文件。

<img src="https://cdn.mazhen.tech/images/202206222053254.png" style="zoom:50%;" />

然后对 **Pro** 进行配置。

| Tab页 | 设置                                             |
| ----- | ------------------------------------------------ |
| 文本  | 勾选“**平滑文本**”。可自定义背景透明度。         |
| 窗口  | **窗口大小**：行 120 列30                        |
| 窗口  | 选择 **将行数限制为**：10000                     |
| shell | **当shell退出时**，选择**当shell完全退出后关闭** |
| 键盘  | 勾选 **将option键当Meta键**                      |
| 高级  | 确认终端为 **xterm-256 color**                   |

## 安装**Xcode Command Line Tools**

**Xcode Command Line Tools** 包含了**clang**编译器，**git**客户端等命令行常用的工具。使用下面的命令安装：

```
xcode-select --install
```

## 安装**Oh My Zsh**

参照 `Oh My ZSH!` 的[官方文档](https://github.com/ohmyzsh/ohmyzsh/wiki)进行安装。

```shell
#确认zsh版本
zsh --version

#执行安装
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如出现连接问题，请在终端设置科学上网。

```shell
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

## 安装和配置 Powerlevel10k

[powerlevel10k](https://github.com/romkatv/powerlevel10k) 是一个Zsh的主题，具体很强的灵活性，并且非常美观。

首先安装 Powerlevel10k 所推荐的字体 Meslo Nerd Font，可以在命令行终端显示一些特殊符号。下载并安装下列字体：

- [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

然后更改终端的字体，在终端的偏好设置的的描述文件中，选择我们使用的 Pro，设置字体为`MesloLGS NF`， 字体大小为 14。

由于我们使用的是Oh My Zsh，可以把Powerlevel10k作为一个主题，安装到Oh My Zsh中：

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

如果遇到网络问题，可以参考上面，设置终端科学上网代理。

编辑Oh My Zsh 的配置文件 **~/.zshrc**，设置主题为Powerlevel10k

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

重新开启终端，按照提示进行Powerlevel10k的样式配置，完成后，我们漂亮的命令行终端配置就大功告成了。

![](https://cdn.mahaoliang.tech/images/202207231529094.png)

如果你在使用**VSCode**，需要在配置文件**settings.json**中设置下面两个配置，你就可以让**VSCode**的终端同样适配Powerlevel10k。

```
 "terminal.integrated.fontSize": 14, 
 "terminal.integrated.fontFamily": "MesloLGS NF"
```



## 安装 zsh-autosuggestions 插件

Oh My Zsh在安装完成后，已经自动配置了git插件。为了在命令行终端更快捷的工作，还可以为Oh My Zsh安装[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 插件。

[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 提供类似于[Fish shell](https://fishshell.com) 自动建议功能，它会根据历史记录，在你键入命令的时候，提供非侵入式的自动建议。

安装[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 的命令：

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

在 `~/.zshrc` 中配置插件：

```bash
plugins=(git zsh-autosuggestions)
```

在键入命令时，会有灰色的提示信息，按 `→` 或 `ctrl-f` 自动完成。是不是非常方便，用过后就完全离不开了。

<img src="https://cdn.mazhen.tech/images/202206261907246.png" style="zoom:67%;" />



## 命令别名配置

使用命令行操作非常快速便捷，但有时候你不知道命令具体干了什么，例如我输入的 `rm *`到底删除了哪些文件。

其实这些命令都有参数，详细的输出该命令影响的文件。但每次都输入这些参数实在太麻烦，我们可以在 Zsh 的配置文件 **~/.zshrc** 中为这些命令设置别名：

```bash
alias mkdir='mkdir -v'
alias mv='mv -v'
alias cp='cp -v'
alias rm='rm -v'
alias ln='ln -v'
```
