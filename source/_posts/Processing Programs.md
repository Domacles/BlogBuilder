---
title: Processing Program
date: 2017-03-13 22:29:29
tags: CG
---

## Processing 简介

**Processing**是一种具有革命前瞻性的新兴计算机语言，它的概念是在电子艺术的环境下介绍程序语言，并将电子艺术的概念介绍给程序设计师。它是 **Java** 语言的延伸，并支持许多现有的 Java 语言架构，不过在语法 (`syntax`) 上简易许多，并具有许多贴心及人性化的设计。**Processing** 可以在 _Windows、MAC OS X、MAC OS 9 、Linux_ 等操作系统上使用。目前最新版本为**Processing 3.3**。以 **Processing** 完成的作品可在个人本机端作用，或以**Java Applets** 的模式外输至网络上发布。

虽然图形用户界面(`GUI`)早在二十年前成为主流，但是基础编程语言的教学到今天仍是以命令行接口为主，学习编程语言为什么要那么枯燥呢？人脑天生擅长空间辨识，图形用户界面利用的正是这种优势，加上它能提供各种实时且鲜明的图像式反馈 (**feedback**)，可以大幅缩短学习曲线，并帮助理解抽象逻辑法则。举例来说，计算机屏幕上的一个像素(**pixel**) 就是一个变量值(**the value of a variable**) 的可视化表现。**Processing**将Java的语法简化并将其运算结果“感官化”，让使用者能很快享有声光兼备的交互式多媒体作品。

**Processing**的源代码是开放的，和近来广受欢迎的 Linux 操作系统、Mozilla浏览器、或Perl语言等一样，用户可依照自己的需要自由裁剪出最合适的使用模式。**Processing**的应用非常丰富，而且它们全部遵守开放源代码的规定，这样的设计大幅增加了整个社群的互动性与学习效率。

## 无意中找到了这个……

话说，我在公司里负责三维模型的处理，而我司的**垃圾代码**不仅经常出Bug，还要什么没什么，基本上只有一个简单的**Mesh**数据结构(OpenMesh)，矩阵计算也只支持 4 * 4 规模的。而我处理三维模型的点线面需要用到多维矩阵。不仅如此，接口调用复杂，一个想法难以用这套软件快速实现……最主要是没有API文档，没有注释，也不给看函数的实现，给的全是**lib**文件！！！

我想写一下**Delaunay三角化算法**的实现也得先从**参数化**开始一步一步实现……

**Processing** 真是及时雨啊。

## 这里有一个3D Delauny Triangulation算法

来自：[www.openprocessing.org - By Tercel](https://www.openprocessing.org/sketch/31295#)，是一个日本人写的代码，我跑了一下效果如下：

![3d_Delaunay](Processing Programs/3d_Delaunay.gif)

## 感谢

+ [openprocessing](https://www.openprocessing.org) 软件的支持
+ [Tercel](https://www.openprocessing.org/sketch/31295#) 的实现
+ [jakkubu](https://www.openprocessing.org/sketch/180760) 的另外一种实现