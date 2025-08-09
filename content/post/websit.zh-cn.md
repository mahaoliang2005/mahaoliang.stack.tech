---
title: "文章发表"
date: 2023-08-31T20:54:03+08:00
draft: false
tags: [article]
categories: [tech]
---

## 生成新的文章

进入到本机网站目录

```
cd documents/works/mahaoliang/mahaoliang.stack.tech
```

在保存本机网站的目录下运行下面的命令，生成一篇文章

```
hugo new post/kubernetes-overview.md
```

## 编辑文章

使用 Typora 打开新建的文章，首先编辑文章的元信息：

```
title: "Kubernetes工作原理概述"
date: 2022-07-22T17:30:11+08:00
draft: false
tags: [docker,linux,kubernetes]
categories: [tech]
```

然后编辑文章内容，图片使用 picGo 上传到图床。

## 将文章发布到 GitHub

在保存本机网站的目录下运行下面的命令，将文章发布到 GitHub

```
git add .
git commit -m "kubernetes"
git push
```

## 网站内容更新

登录到腾讯云主机，

```
ssh guangzhou-tencent
```

进入 `~/works/mahaoliang.stack` 目录，执行命令：

```
git pull
```

完成

