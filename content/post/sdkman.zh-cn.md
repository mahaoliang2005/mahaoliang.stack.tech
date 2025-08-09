---
title: "Sdkman 的使用"
date: 2023-06-24T18:08:17+08:00
draft: false
tags: [java]
categories: [tech]
---

[SDKMAN！](https://sdkman.io/)是用于管理多个软件开发工具包的并行版本的工具。

![sdkman](https://cdn.mahaoliang.tech/images/202306241814580.svg)

可安装的[软件列表](https://sdkman.io/sdks) 

* 列出当前可安装的软件列表
```
sdk list
```

* 列出当前可安装的 Java 版本列表

```
sdk list java
```

* 安装指定的版本

```
sdk install java 17.0.7-tem
```

* 查看当前使用的版本

```
sdk current 
sdk current java
```

* 设置缺失使用的版本

```
sdk default java 17.0.7-tem
```

* 设置当前 shell session 使用的版本

```
sdk use java 8.0.332-tem   
```

* 更新软件仓库

```
sdk update
```

* sdk 软件本身的更新

```
sdk selfupdate
```
