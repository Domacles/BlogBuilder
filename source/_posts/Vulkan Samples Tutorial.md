---
title: Vulkan Samples Tutorial
date: 2017-03-19 19:42:46
tags:
---

<!-- TOC -->

- [Introduction](#introduction)
- [Instance](#instance)
    - [Vulkan Instance](#vulkan-instance)
    - [vkCreateInstance](#vkcreateinstance)
    - [VkInstanceCreateInfo](#vkinstancecreateinfo)
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

相关代码在下载的Vulkan SDK中的**Samples**文件夹中，在使用cmake构建工程之前，请先阅读**Documentation/vulkan_samples.html**文件来了解如何构建时所需要的配置。在此不再赘述。

本文的最终目标是初步理解Vulkan API的使用并创建出显示一个立方体的程序，效果如下：

![Sample](Vulkans Samples Tutorial/sample.png)

---

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
---

## Instance

本章节的代码在`01-init_instance.cpp`文件中。

### Vulkan Instance

Vulkan API使用`vkInstance`来存储程序的预先设置的状态(`per-application state`)，所有的应用程序在使用Vulkan API进行操作之前都必须需要创建**Vulkan Instance**。

Vulkan程序的基础结构如下：
![Vulkan Instances](Vulkan Samples Tutorial/Vulkan Instances.png)

上图告诉我们：Vulkan程序会被连接到一个Vulkan库上，这个库被称为**Loader**。应用程序需要通过创建Vulkan Instance来对loader进行初始化。loader之后会对更底层的显卡驱动进行加载和初始化。

> 上图中，有许多**Layers**被loader加载，这些Layer通常被用作程序校验的，例如对程序的正常运行进行错误检查。在Vulkan中，设备驱动程序变得比在使用OpenGL时更加轻量级化，这是因为在OpenGL中的一些如校验功能等被Layers所代替。Layers是可配置的，也可以在每次创建程序时选择加载。Vulkan Layers不再本文的讨论范围之内。进一步了解请到LunarXchange的Layers章节进行了解和学习。

### vkCreateInstance

在`01-init_instance.cpp`文件中，我们找到下面这一句：
```
res = vkCreateInstance(&inst_info, NULL, &inst);
```
`vkCreateInstance`函数的原型如下：
```
VKAPI_ATTR VkResult VKAPI_CALL vkCreateInstance(
    const VkInstanceCreateInfo*                 pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkInstance*                                 pInstance);
```
在`vulkan.h`头文件中查看该函数定义，我们可以得知：
+ `vkResult`是一个枚举类型，查看其定义可知，它定义了一系列vulkan的状态，如：`VK_SUCCESS`，`VK_NOT_READY`，`VK_TIMEOUT`,`VK_EVENT_SET` ，`VK_EVENT_RESET`，`VK_INCOMPLETE`等。我们可以通过该返回值来判断我们进行的vulkan操作是否得到我们需要的状态，并针对该状态进行处理。
+ `VkInstanceCreateInfo`是一个结构体类型，查看其定义可知，该结构体中用来存储创建Vulkan Instance所必须的一些设置。
+ `VkAllocationCallbacks`也是一个结构体类型，但通过查看其定义，它其中存放的是一些**函数指针**(不支持lambda表达式，但我们可以使用`std::function::target`函数获取lambda表达式的函数指针，具体用法可以参照[cppreference.com - target](http://en.cppreference.com/w/cpp/utility/functional/function/target)的使用方法)，这些函数作为回调函数供`vkCreateinstance`函数使用。该函数用于实现应用程序自己的内存分配策略，如果没有设置，则vulkan会执行其默认的内存分配策略。在本小节的`01-init_instance.cpp`文件中并没有使用该特性，因此调用位置设置的值为`NULL`。
+ `VkInstance`是一个Vulkan Instance类型，该参数是一个返回值，当Vulkan Instance创建成功后该返回值有值。由于该实例并不是一个我们可见的自定义类型，当使用它时不应当去尝试对它**解除引用(de-reference)**。

> 注意：`vkCreateInstance`函数为我们展示了许多Vulkan函数的定义方法，返回值为操作完成的状态，参数中有输入的设置变量，数据变量，也有输出的实例。

### VkInstanceCreateInfo

接下来我们查看一下`VkInstanceCreateInfo`类型的定义：
```
typedef struct VkInstanceCreateInfo {
    VkStructureType             sType;
    const void*                 pNext;
    VkInstanceCreateFlags       flags;
    const VkApplicationInfo*    pApplicationInfo;
    uint32_t                    enabledLayerCount;
    const char* const*          ppEnabledLayerNames;
    uint32_t                    enabledExtensionCount;
    const char* const*          ppEnabledExtensionNames;
} VkInstanceCreateInfo;
```




















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


