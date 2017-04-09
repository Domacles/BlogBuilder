---
title: Vulkan Samples Tutorial 2
date: 2017-04-07 10:18:23
tags: Vulkan
---

<!-- TOC -->

- [Swapchain](#swapchain)
    - [Vulkan and the Windowing System](#vulkan-and-the-windowing-system)
    - [Revisiting Instance and Device Extensions](#revisiting-instance-and-device-extensions)
    - [Queue Family and Present](#queue-family-and-present)
    - [Swapchain Create Info](#swapchain-create-info)
    - [Different Queue Families for Graphics and Present](#different-queue-families-for-graphics-and-present)
    - [Create Swapchain](#create-swapchain)
    - [Create Image Views](#create-image-views)
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
+ VK_USE_PLATFORM_XCB_KHR - X Window System, using the XCB library (Apples)
+ VK_USE_PLATFORM_XLIB_KHR - X Window System, using the Xlib library (Apples)
**KHR** 命名后缀表示扩展是按照 **Khronos** 组织的规范实现的。

Surface Abstraction

Vulkan 使用 `VkSurfaceKHR` 对象来作为本地平台显示层或者窗口的抽象化对象。该对象是在**VK_KHR_surface**扩展中定义的。WSI扩展的作用是创建，操纵或者销毁显示层对象(surface objects)。

### Revisiting Instance and Device Extensions

在前面的几个章节中，我们推迟了扩展的设置，现在需要我们回顾之前 Instance 和 Device 的扩展设置，从而能够让我们学会如何启动WSI扩展。

Instance Extensions

使用WSI扩展的首要步骤是，启动表示层扩展(surface extension)。查看本章样例中使用的`init_instance_extension_names()`函数的实现代码，我们发现样例是将`VK_KHR_SURFACE_EXTENSION_NAME`添加到Instance扩展列表：
```
void init_instance_extension_names(struct sample_info &info) {
    info.instance_extension_names.push_back(VK_KHR_SURFACE_EXTENSION_NAME);
#ifdef __ANDROID__
    info.instance_extension_names.push_back(VK_KHR_ANDROID_SURFACE_EXTENSION_NAME);
#elif defined(_WIN32)
    info.instance_extension_names.push_back(VK_KHR_WIN32_SURFACE_EXTENSION_NAME);
#else
    info.instance_extension_names.push_back(VK_KHR_XCB_SURFACE_EXTENSION_NAME);
#endif
}
```
该函数不仅添加了通用Surface扩展，还针对其他平台添加对应的扩展。比如，针对Win32平台会将`VK_KHR_WIN32_SURFACE_EXTENSION_NAME`添加到Instance扩展列表中。

这些扩展将会在Instance创建时加载，加载的实现代码可以在`init_instance()`中进行查看。
> 注意：在`init_instance()`中我们看到`instance_extension_names`列表是通过传指针数组将内容的地址设置到`ppEnabledExtensionNames`上的。

Device Extensions

Swapchain 是一个图像缓存的列表，GPU向其中输入图像，该列表的图像会被呈现到显示输出设备。每当GPU向其中写入图像数据，device-level 扩展就会开始处理 Swapchain。所以，在进行 device 初始化之前需要指定device扩展，在`init_device_extension_names()`函数中，我们看到使用的device扩展是`VK_KHR_SWAPCHAIN_EXTENSION_NAME`:
```
void init_device_extension_names(struct sample_info &info) {
    info.device_extension_names.push_back(VK_KHR_SWAPCHAIN_EXTENSION_NAME);
}
```
这些扩展将在创建Device时进行加载，加载的实现代码可以在`init_device()`中进行查看，与`init_instance()`类似。

Instance & Device Extensions recap：
+ 本节样例使用 `init_instance_extension_names()` 函数来加载常用的surface扩展，并将平台相关的对应扩展也加到了instance扩展列表中了。
+ 本节样例使用 `init_device_extension_names()` 函数来加载一个Swapchain设备扩展。

### Queue Family and Present

**Present** 操作，就是使一个Swapchain图像缓冲放到物理显示设备上的操作。当我们的应用程序需要显示图像，那就需要向某一个GPU设备队列发送一个**呈现**(Present)请求（具体方法是调用`vkQueuePresentKHR()`实现的）但接受请求的GPU队列需要能够支持present请求，或者支持graphics和present请求。下面的代码就展示了如何找到一个支持graphics和present操作的GPU队列：
```
// Iterate over each queue to learn whether it supports presenting:
VkBool32 *pSupportsPresent =
    (VkBool32 *)malloc(info.queue_family_count * sizeof(VkBool32));
for (uint32_t i = 0; i < info.queue_family_count; i++) {
    vkGetPhysicalDeviceSurfaceSupportKHR(info.gpus[0], i, info.surface,
                                         &pSupportsPresent[i]);
}

// Search for a graphics and a present queue in the array of queue
// families, try to find one that supports both
info.graphics_queue_family_index = UINT32_MAX;
info.present_queue_family_index = UINT32_MAX;
for (uint32_t i = 0; i < info.queue_family_count; ++i) {
    if ((info.queue_props[i].queueFlags & VK_QUEUE_GRAPHICS_BIT) != 0) {
        if (info.graphics_queue_family_index == UINT32_MAX)
            info.graphics_queue_family_index = i;

        if (pSupportsPresent[i] == VK_TRUE) {
            info.graphics_queue_family_index = i;
            info.present_queue_family_index = i;
            break;
        }
    }
}

if (info.present_queue_family_index == UINT32_MAX) {
    // If didn't find a queue that supports both graphics and present, then
    // find a separate present queue.
    for (size_t i = 0; i < info.queue_family_count; ++i)
        if (pSupportsPresent[i] == VK_TRUE) {
            info.present_queue_family_index = i;
            break;
        }
}
free(pSupportsPresent);
```
上面这份代码，再次使用在这之前便已获取的变量`info.queue_family_count`，通过调用`vkGetPhysicalDeviceSurfaceSupportKHR()`函数来获取每一个queue family的是否支持surface的标志。然后遍历所有的queue family，来寻找同时支持present和graphics的GPU队列族。最后，如果发现没有找到同时支持present和graphics的GPU队列族，则去寻找一个支持present的GPU队列族，将present和graphics功能分到两个GPU queue family上。接下来的代码，会从`graphics_queue_family_index`获取执行图形命令的队列编号，从`present_queue_family_index`获取执行呈现操作的队列编号。

### Swapchain Create Info

### Different Queue Families for Graphics and Present

### Create Swapchain

### Create Image Views

---

## Depth Buffer

## Uniform Buffer


---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~