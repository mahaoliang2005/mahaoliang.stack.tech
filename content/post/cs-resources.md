---
title: "大学生计算机专业学习资源不完全列表"
date: 2024-08-24T15:51:30+08:00
draft: false
tags: [linux]
categories: [tech]
---

![6](https://cdn.mahaoliang.tech/2024/202408241600736.webp)

## 计算机系统与组成原理

* 极客时间：[深入浅出计算机组成原理](https://time.geekbang.org/column/intro/100026001)
* [Computer Systems: A Programmer's Perspective](http://csapp.cs.cmu.edu/3e/home.html) 从程序员的角度学习计算机系统，了解计算机系统的各个方面，包括硬件、操作系统、编译器和网络。这本书涵盖了数据表示、C 语言程序的机器级表示、处理器架构、程序优化、内存层次结构、链接、异常控制流（异常、中断、进程和 Unix 信号）、虚拟内存和内存管理、系统级 I/O、基本的网络编程和并发编程等概念。这些概念由一系列有趣且实践性强的实验室作业支持。
* [Computer Systems: A programmer's Perspective 视频课](https://www.youtube.com/playlist?list=PLyboo2CCDSWnhzzzzDQ3OBPrRiIjl-aIE)
* [Computer Science from the Bottom Up](https://bottomupcs.com/index.html) 采用“从下到上”的方法，从最基础的二进制、数据表示开始，逐步深入计算机内部工作原理，目的是帮助读者真正掌握计算机科学的基础知识。
* [Putting the “You” in CPU](https://cpu.land/)  深入探讨了计算机系统的工作原理，包括 CPU 的基本操作、系统调用、多任务处理、内存管理以及程序的执行过程。
* [编码](https://book.douban.com/subject/4822685/)  从二进制编码、数据表示到计算机体系结构、操作系统等多个重要主题，从根本上理解计算机的工作原理。
* [漫画计算机原理](https://book.douban.com/subject/35658408/)
* [趣话计算机底层技术](https://book.douban.com/subject/36428782/)
* [计算机底层的秘密](https://book.douban.com/subject/36370606/)
* [穿越计算机的迷雾](https://book.douban.com/subject/30198087/)
* [嵌入式 C 语言自我修养](https://book.douban.com/subject/35446929/)

## C 语言

* [征服 C 指针](https://book.douban.com/subject/35384099/) 彻底理解和掌握指针的各种用法和技巧
* [C 专家编程](https://book.douban.com/subject/35218533/) Sun 公司编译器和 OS 核心开发团队成员，对 C 的历史、语言特性、声明、数组、指针、链接、运行时、内存等问题进行了细致的讲解和深入的分析
* [C from Scratch](https://github.com/theokwebb/C-from-Scratch) 一个学习 C 语言的从零开始的路线图，包括推荐的课程、项目和资源，以及进阶到 x86-64 汇编语言和操作系统内部的指导。
* 极客时间：[深入 C 语言和程序运行原理](https://time.geekbang.org/column/intro/100100701)
* [cdecl](https://cdecl.org/) 将C语言声明转换为英文描述，例如将这样复杂的声明 `void (\*signal(int, void (\*)(int)))(int)` 转换为文字描述：“declare signal as function (int, pointer to function (int) returning void) returning pointer to function (int) returning void”

## 程序运行原理

*  [Online Compiler, Visual Debugger](https://pythontutor.com/) 独特的逐步可视化调试工具，强烈推荐！
* [程序是怎样跑起来的](https://book.douban.com/subject/26365491/)
* [程序员的自我修养：链接、装载与库](https://book.douban.com/subject/3652388/)
* 如何从对象文件中导入和执行代码  [part1](https://blog.cloudflare.com/how-to-execute-an-object-file-part-1/) [part2](https://blog.cloudflare.com/how-to-execute-an-object-file-part-2/) [part3](https://blog.cloudflare.com/how-to-execute-an-object-file-part-3/)
* [x86/x64 CPU architecture: the stack & stack frames](https://yuriygeorgiev.com/2024/02/19/x86-64-cpu-architecture-the-stack/)  x86/x64 CPU 架构中的栈（Stack）及其工作机制，包括栈的数据结构特性、CPU 中栈的管理、栈与堆的区别、栈帧的创建与销毁，以及栈的性能优势。
* [Driving Compilers](https://fabiensanglard.net/dc/) 关于如何使用编译器创建可执行文件的深入知识，涵盖编译器驱动程序、预处理器 cpp、编译器 cc、链接器 ld 以及 Linux 加载器的概念。

## Linux 使用

* 极客时间：[Linux 实战技能 100 讲](https://time.geekbang.org/course/intro/100029601)
* [Efficient Linux at the Command Line](https://www.oreilly.com/library/view/efficient-linux-at/9781098113391/)
* [像黑客一样使用命令行](https://selfhostedserver.com/usingcli)
* [Linux Foundation](https://www.linuxfoundation.org/) 的认证考试 [LFCA](https://training.linuxfoundation.org/certification/certified-it-associate/#) 和 [LFCS](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/#)
* [Learning Modern Linux](https://www.oreilly.com/library/view/learning-modern-linux/9781098108939/)
* [Linux From Scratch](https://www.linuxfromscratch.org/lfs/) step-by-step instructions for building your own customized Linux system entirely from source.

## Linux 内核

* [Linux 是怎么工作的](https://book.douban.com/subject/35768243/)
* [Linux 技术内幕](https://book.douban.com/subject/26931513/)
* [Linux 内核设计与实现](https://book.douban.com/subject/6097773/) Linux Kernel Development
* [深入理解Linux进程与内存](https://book.douban.com/subject/37015972/)
* 极客时间：[Linux 内核技术实战课](https://time.geekbang.org/column/intro/100058001)
* 极客时间：[编程高手必学的内存知识](https://time.geekbang.org/column/intro/100094901)
* 极客时间：[容器实战高手课](https://time.geekbang.org/column/intro/100063801)
* [深入理解 Linux 网络](https://book.douban.com/subject/35922722/)
* [交互式的 Linux 内核地图](https://makelinux.github.io/kernel/map/)

## Linux 系统编程

* [Linux/UNIX系统编程手册](https://book.douban.com/subject/25809330/) The Linux Programming Interface: A Linux and UNIX System Programming Handbook
* [UNIX 环境高级编程](https://book.douban.com/subject/25900403/) Advanced Programming in the UNIX Environment
* [CS 341: System Programming ](https://github.com/illinois-cs241/coursebook) 伊利诺伊大学香槟分校 CS 341 课程使用，介绍 C 语言和 Linux 系统编程知识。

## 网络

* [趣谈网络协议](https://book.douban.com/subject/35013753/)
* 极客时间：[Web 协议详解与抓包实战](https://time.geekbang.org/course/intro/100026801)
* [图解 TCP/IP](https://book.douban.com/subject/24737674/)
* [图解 HTTP](https://book.douban.com/subject/25863515/)
* [网络是怎样连接的](https://book.douban.com/subject/26941639/)

## 数据结构和算法

* 极客时间：[数据结构与算法之美](https://time.geekbang.org/column/intro/100017301)
* 极客时间：[算法面试通关 40 讲](https://time.geekbang.org/course/intro/100019701)
* 极客时间：[常用算法 25 讲](https://time.geekbang.org/opencourse/intro/100057601)
* 极客时间：算法训练营
* [Hello 算法](https://github.com/krahets/hello-algo) 动画图解、一键运行的数据结构与算法教程
* [通过动画可视化数据结构和算法](https://visualgo.net/zh)

## 算法刷题

* [Leetcode](https://leetcode.com/) 一个广受欢迎的在线编程题库
* [Neetcode](https://neetcode.io/) 另一个在线编程练习平台
* [代码随想录](https://github.com/youngyangyang04/leetcode-master) LeetCode 刷题攻略
* [算法通关手册](https://github.com/itcharge/LeetCode-Py) 850+ 道「LeetCode 题目」详细解析

## 综合

* [计算机自学指南](https://csdiy.wiki/) ([GitHub 仓库](https://github.com/PKUFlyingPig/cs-self-learning))
* YouTube 视频课：[Crash Course Computer Science Preview](https://www.youtube.com/watch?v=tpIctyqH29Q&list=PL8dPuuaLjXtNlUrzyH5r6jN9ulIgZBpdo)
* [计算机教育中缺失的一课](https://missing-semester-cn.github.io/)
* [Developer Roadmaps](https://roadmap.sh/) 为开发者提供学习路线图和指南
* [Online Coding Classes – For Beginners](https://www.freecodecamp.org/news/online-coding-classes-for-beginners-2022-guide/)  3000 小时的免费课程，涵盖了编程涉及到的方方面面

## 交互式教程

* [Grep by example](https://antonz.org/grep-by-example/) 如何使用命令行工具 grep 进行文本搜索的交互式指南
* [Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN) 一个交互式的在线教程，帮助用户学习并练习 Git 的基本使用方法

## 在线课程

* [educative](https://www.educative.io/) 为开发者提供交互式在线课程，重点关注技术领域的知识与技能
* [edX](https://www.edx.org/) 由麻省理工学院（MIT）和哈佛大学共同创立的在线教育平台
* [exercism](https://exercism.org/) 专注于通过有趣且具有挑战性的练习问题、支持建设性同行评审机制来促进积极参与和技能提升，从而培养对各种现代计算范式的熟练掌握。

## 技术面试

* [Cracking the coding interview book](https://www.amazon.com/Cracking-Coding-Interview-Programming-Questions/dp/0984782850) 一本深受程序员喜爱的面试指南书
* [编程面试大学](https://github.com/jwasham/coding-interview-university/blob/main/translations/README-cn.md) 涵盖了算法、数据结构、面试准备和工作机会等主题，帮助你准备大公司的技术面试
* [interviewing.io](https://interviewing.io/) 一个提供模拟技术面试的平台
* [Pramp](https://www.pramp.com/) 一个模拟面试平台
* [Meetapro](https://www.meetapro.com/) 一个可以找到专业人士进行模拟面试的网站
* [PPResume](https://ppresume.com/) 一个基于 LaTeX 的简历生成器，目标是帮助人们在几分钟内创建一份精美的简历，并提供极高质量的排版和 PDF 输出。
## 大语言模型

* [Learn Prompting](https://learnprompting.org/zh-Hans/docs/intro) 一个开源的、多元化社区构建的课程，旨在提供完整、公正的提示工程知识。
* [提示工程指南](https://www.promptingguide.ai/zh) 介绍大语言模型（LLM）相关的论文研究、学习指南、模型、讲座、参考资料、大语言模型能力及其与其他工具的对接。
* [面向开发者的大模型手册](https://datawhalechina.github.io/llm-cookbook/) 基于吴恩达大模型系列课程的翻译和复现项目，涵盖了从 Prompt Engineering 到 RAG 开发的全部流程，为国内开发者提供了学习和入门 LLM 相关项目的方式。
* [LLM 应用开发实践笔记](https://aitutor.liduos.com/) 作者在学习基于大语言模型的应用开发过程中总结出来的经验和方法，包括理论学习和代码实践两部分。
* [动手学大模型应用开发](https://datawhalechina.github.io/llm-universe/) 面向小白开发者的大模型应用开发教程，基于阿里云服务器，结合个人知识库助手项目，通过一个课程完成大模型开发的重点入门。

## iOS 开发

* [iOS & Swift - The Complete iOS App Development Bootcamp](https://www.udemy.com/course/ios-13-app-development-bootcamp/)
* [The 100 Days of SwiftUI](https://www.hackingwithswift.com/100/swiftui)
* [Stanford CS193p - Developing Apps for iOS](https://cs193p.sites.stanford.edu/2023)
* [iOS and SwiftUI for Beginners](https://www.kodeco.com/ios/paths/learn)
* [Meta iOS Developer](https://www.coursera.org/professional-certificates/meta-ios-developer)
* [Develop in Swift Tutorials](https://developer.apple.com/tutorials/develop-in-swift-tutorials) 苹果官方教程
* [SwiftUI Tutorials](https://developer.apple.com/tutorials/swiftui) 苹果官方教程

## 计算机科学史

* [信息简史](https://book.douban.com/subject/25752043/)
