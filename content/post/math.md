---
title: "如何用命令行绘制数学函数"
date: 2023-09-15T15:07:58+08:00
draft: false
tags: [math]
categories: [tech]
---
## 文本准备

- 要创建一个以 .py 扩展名结尾的文本文件，你可以按照以下步骤进行操作：

  - 打开文本编辑器。
  
    你可以使用操作系统自带的文本编辑器（如记事本、TextEdit等），或者使用专业的代码编辑器（如Visual Studio Code、Sublime Text、Atom等）。

  - 在文本编辑器中创建一个新文件。

    将你的Python代码复制粘贴到新文件中。
保存文件时，指定文件名并确保使用 .py 作为文件的扩展名。
例如，你可以将文件命名为 plot_example.py。

- 代码示例（画的是一个y=x^2的图）：

```import matplotlib.pyplot as plt
import numpy as np
# 准备数据
x = np.linspace(-10, 10, 100)  # 在 -10 到 10 之间生成 100 个均匀分布的点

# 计算对应的 y 值
y = x ** 2

# 创建图形并设置标题
plt.figure()
plt.title("Plot Example")

# 绘制折线图
plt.plot(x, y)

# 显示图形
plt.show()
repr 
```

## 运行

- 我在Mac上用的是**visual studio code**，
点击文件，用VS code打开。

![图片不见啦](https://cdn.mahaoliang.tech/images/202309151521049.png)

- 打开后，假如没装python的话，会显示要install python 编译器，下载就可以运行了了

![图片不见啦](https://cdn.mahaoliang.tech/images/202309151527687.png)
