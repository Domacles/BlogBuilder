---
title: Vulkan Samples Tutorial
date: 2017-03-19 19:42:46
tags:
---

<!-- TOC -->

- [Introduction](#introduction)
- [Instance](#instance)
    - [Vulkan Instance](#vulkan-instance)
- [Enumerate Devices](#enumerate-devices)
- [Device](#device)
- [Command Buffer](#command-buffer)
- [Swapchain](#swapchain)
- [Depth Buffer](#depth-buffer)
- [Uniform Buffer](#uniform-buffer)
- [Pipeline Layout](#pipeline-layout)
- [Descriptor Set](#descriptor-set)
- [Render Pass](#render-pass)
- [Shaders](#shaders)
- [Framebuffers](#framebuffers)
- [Vertex Buffer](#vertex-buffer)
- [Pipeline](#pipeline)
- [Draw Cube](#draw-cube)

<!-- /TOC -->

本文会通过几个章节告诉你们如何一步一步地创建一个简单的Vulkan程序，每一章节对应着Vulkan程序运行的每一部分过程。

相关代码在下载的Vulkan SDK中的**Samples**文件夹中，在使用cmake构建工程之前，请先阅读**README.md**文件来了解如何构建时所需要的配置。在此不再赘述。

本文的最终目标是初步理解Vulkan API的使用并创建出显示一个立方体的程序，效果如下：

![Sample](Vulkans Samples Tutorial/sample.png)


## Introduction

每个章节的代码使用下面的格式，目的是为了能够使大家能够了解该章节的主题。
```
init_instance(info, sample_title);
init_enumerate_device(info);
init_window_size(info, 500, 500);
init_connection(info);
init_window(info);
init_swapchain_extension(info);
init_device(info);
...
/* VULKAN_KEY_START */

... code of interest

/* VULKAN_KEY_END */
...
destroy_device(info);
destroy_window(info);
destroy_instance(info);
```
## Instance

本章节的代码在`01-init_instance.cpp`文件中。

### Vulkan Instance

Vulkan API使用`vkInstance`来存储程序的预先设置的状态(`per-application state`)，所有的应用程序在使用Vulkan API进行操作之前都必须需要创建**Vulkan Instance**。

Vulkan程序的基础结构如下：
![Vulkan Instances](Vulkan Samples Tutorial/Vulkan Instances.png)

上图告诉我们：Vulkan程序会被连接到一个Vulkan库上，这个库被称为**Loader**。应用程序需要通过创建Vulkan Instance来对loader进行初始化。loader之后会对更底层的显卡驱动进行加载和初始化。

> 上图中，有许多**Layers**被loader加载，这些Layer通常被用作程序校验的，例如对程序的正常运行进行错误检查。在Vulkan中，设备驱动程序变得比在使用OpenGL时更加轻量级化，这是因为在OpenGL中的一些如校验功能等被Layers所代替。Layers是可配置的，也可以在每次创建程序时选择加载。Vulkan Layers不再本文的讨论范围之内。进一步了解请到LunarXchange的Layers章节进行了解和学习。



## Enumerate Devices

## Device

## Command Buffer

## Swapchain

## Depth Buffer

## Uniform Buffer

## Pipeline Layout

## Descriptor Set

## Render Pass

## Shaders

## Framebuffers

## Vertex Buffer

## Pipeline

## Draw Cube












































本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~


