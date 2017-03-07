---
title: Vulkan 简介与使用
date: 2017-03-08 02:18:13
tags: Vulkan
---

<!-- TOC -->

- [Vulkan介绍](#vulkan介绍)
- [使用Vulkan之前的准备](#使用vulkan之前的准备)
- [Vulkan SDK安装与使用指南](#vulkan-sdk安装与使用指南)
    - [Vulkan SDK的下载](#vulkan-sdk的下载)
    - [Vulkan SDK的安装](#vulkan-sdk的安装)
- [感谢](#感谢)

<!-- /TOC -->

## Vulkan介绍

**Vulkan** 是一个低CPU开销、跨平台的2D和3D绘图与计算的应用程序接口（`API`），最早由科纳斯组织（`Khronos`）在2015年游戏开发者大会（`GDC`）上发表。科纳斯最先把`Vulkan API`称为“**次世代OpenGL行动**”（next generation OpenGL initiative）或“**glNext**”，但在正式宣布Vulkan之后这些名字就没有再使用了。

`Vulkan`基于`Mantle`而构建，这是由于 **AMD** 将 **Mantle API** 捐赠给科纳斯组织，给予该组织开发底层`API`的基础，使其成为行业标准。

与`OpenGL`类似，`Vulkan`针对**全平台即时3D程序**（如电子游戏和交互媒体）设计，并提供**高性能**与**更均衡**的CPU/GPU使用，这也是`Direct3D 12`和AMD的`Mantle`的目标。与`Direct3D`（12版之前）和`OpenGL`的其他主要区别是，`Vulkan`是一个底层`API`，并且能执行并行任务。除此之外，`Vulkan`还能更好地分配多个CPU核心的使用。

## 使用Vulkan之前的准备

* PC配备一块支持Vulkan的显卡，并安装支持Vulkan的显卡驱动(NVDIA, AMD, Inter)
* 具备使用C++的经验(RAII, Class, STL...)
* 支持C++11的编译器(VS 2013+, GCC 4.8+, Clang...)
* 有一定的计算机图形学基础，了解一些OpenGL渲染运行机制

注意：实际上**Vulkan**也提供了**C Vulkan API**， 如果使用windows系统，安装**LUNAR vulkan SDK**后，可以按照使用说明建立VS工程：`Vulkan Program`，`Vulkan Windowed Program`，`Vulkan C++ Program`，`Vulkan C++ Windowed Program`。

## Vulkan SDK安装与使用指南

LunarG Vulkan SDK 提供了一套用于创建、运行并调试 Vulkan 应用程序的开发和运行组件。但该 SDK 并不提供支持Vulkan的**驱动程序**，驱动程序需要从你所使用的显卡提供商那进行获取并安装。

在使用 SDK 之前需要了解下面这7个术语：

Term    | Description
--------|-----------------------------------------------------------------
ICD     | Installable Client Driver — A Vulkan-compatible display driver（安装在PC上的支持Vulkan的驱动程序）
GLSL    | OpenGL Shading Language（OpenGL的着色器语言，Vulkan支持）
Layer   | A library designed to work as a plug-in for the loader. It usually serves to provide validation and debugging functionality to applications（一种可用于加载器的插件库，这个库可以为应用程序的图形渲染提供检查和调试功能，插件）
Loader  | A library which implements the Vulkan API entry points and manages layers, extensions, and drivers. It can be found in the SDK, as well as independent hardware vendor driver installs（独立于硬件驱动安装的软件库，是用于实现Vulkan API入口，管理插件、扩展和驱动程序。）
SPIR-V  | Standard Portable Intermediate Representation — A cross-API intermediate language that natively represents parallel compute and graphics programs（SPIR-V 中间语言(比如GLSL)，用于程序与硬件进行交流的语言）
Vulkan  | A low overhead, explicit graphics API developed by the Khronos Group and member companies（一个由科纳斯组织及其成员设计的底层图形API）
WSI     | Window System Integration（窗口系统集组件）

### Vulkan SDK的下载

点击[Vulkan.LunarG.com](Vulkan.LunarG.com)的“Download Vulkan Tools for Windows”下载其中的`VulkanSDK version Installer.exe`程序，打开安装即可。如果你使用的是Linux系统则需要到该网站的Linux下载安装页面进行安装。

### Vulkan SDK的安装

Windows系统安装完成之后需要为Vulkan SDK配置环境变量。

* 在变量中添加`VULKAN_SDK`变量，变量的值为安装的Vulkan SDK的位置，SDK的默认安装位置在`C:\VulkanSDK\version`下，version是版本号。
* 在变量中添加`VK_SDK_PATH`变量，变量的值与`VULKAN_SDK`相同。
* 在环境变量（`PATH`）中添加：`$VULKAN_SDK\Bin`（32位操作系统则添加`$VULKAN_SDK\Bin32`）。

Vulkan loader程序，即`vulkan-1.dll`，在32位的系统上需要将32位版本的dll安装到`C:\Windows\System32`中，64位的系统上需要将64位版本的dll安装到`C:\Windows\SysWOW64`中。

>注意：实际上上面的步骤只是需要检查一下，如果Vulkan SDK的安装软件并没有设置这些环境变量则需要手动加上。




































## 感谢

* [Getting Started with the Vulkan SDK - LunarG](https://vulkan.lunarg.com/doc/sdk/1.0.26.0/windows/getting_started.html)