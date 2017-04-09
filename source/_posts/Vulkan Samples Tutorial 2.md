---
title: Vulkan Samples Tutorial 2
date: 2017-04-07 10:18:23
tags: Vulkan
---

<!-- TOC -->

- [Swapchain](#swapchain)
    - [Vulkan and the Windowing System](#vulkan-and-the-windowing-system)
    - [Revisiting Instance and Device Extensions](#revisiting-instance-and-device-extensions)
- [Depth Buffer](#depth-buffer)
- [Uniform Buffer](#uniform-buffer)

<!-- /TOC -->

## Swapchain

本章节的代码在文件 `05-init_swapchain.cpp` 中。

**Swapchain** 即帧缓存机制，它是使用了一系列的帧缓存区用来保证帧刷新率稳定的技术。帧缓存区通常放在显卡内存中，但也可以放在内存中。Swapchain 需要通过图形API开启并使用，未使用 Swapchain 会导致刷新不一致等问题。每个 swap chain 至少有两个缓冲区，一个作为屏幕显示缓冲区，另外一个作为后台缓冲区。两个缓冲区一般被称为 **presentation** 和 **swapping** 。

本小节讲述了如何创建**swapchain**。首先需要做的就是创建用来渲染的图形缓冲区(image buffers)。
![Swapchain](Vulkan Samples Tutorial 2/Swapchain.png)
> 上图展示了Swapchain与其他各部分之间的联系，图中的其他部分与下面所讲的内容关系也十分密切，需要注意。

### Vulkan and the Windowing System

与其他的图形API相似的是，Vulkan将有关于窗口系统的API与图形核心API分离开了。在Vulkan中，窗口系统的细节是通过 **WSI** 扩展(Window System Integration extensions)呈献给开发者的。我们可以从Vulkan的设计规范中找到与WSI相关的扩展的文档。在[LunarG LunarXchange website](https://vulkan.lunarg.com/)和[Khronos Vulkan Registry](https://www.khronos.org/registry/vulkan/)这两个地方可以找到Vulkan API规范和发布的扩展的使用规范。

WSI扩展包含了对多种平台的支持。使用WSI扩展是通过定义下面几个宏定义来实现的：
+ VK_USE_PLATFORM_ANDROID_KHR - Android
+ VK_USE_PLATFORM_MIR_KHR - Mir
+ VK_USE_PLATFORM_WAYLAND_KHR - Wayland
+ VK_USE_PLATFORM_WIN32_KHR - Microsoft Windows
+ VK_USE_PLATFORM_XCB_KHR - X Window System, using the XCB library
+ VK_USE_PLATFORM_XLIB_KHR - X Window System, using the Xlib library
**KHR** 命名后缀表示扩展是按照 **Khronos** 组织的规范实现的。

Surface Abstraction

Vulkan 使用 `VkSurfaceKHR` 对象来作为本地平台显示层或者窗口的抽象化对象。该对象是在**VK_KHR_surface**扩展中定义的。WSI扩展的作用是创建，操纵或者销毁显示层对象(surface objects)。

### Revisiting Instance and Device Extensions



---

## Depth Buffer

## Uniform Buffer


---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~