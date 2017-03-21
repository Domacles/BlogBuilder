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
    - [VkApplicationInfo](#vkapplicationinfo)
    - [Back to the Code](#back-to-the-code)
- [Enumerate Devices](#enumerate-devices)
    - [vkEnumeratePhysicalDevices](#vkenumeratephysicaldevices)
    - [The Samples_info Structure](#the-samples_info-structure)
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
该结构体的前两个属性，会在许多Vulkan `CreateInfo`中进行定义。
+ `sType`是一个枚举类型，表明当前`CreateInfo`是个什么类型的结构体，虽然看似与`VkInstanceCreateInfo`的命名作用重合，在`CreateInfo`结构体声明自己的类型有两个原因：
    - 驱动程序(driver)，校验插件(validation layer)或者其他定义的结构可以方便地进行类型检验，如果不符合期望类型则会停止执行下面的操作。
    - 该结构体可以被一个无类型指针(void *)传递到一个使用未定义类型指针的函数中，在该函数中进行类型转换后可以方便进行类型检验。比如，一个驱动程序支持一个扩展来创建vulkan实例，该驱动程序就会沿着`pNext`指向的`CreateInfo`链表查找指定类型的结构体，以满足插件的需要。
+ `pNext`是一个无类型指针，它有时候会被用于指向一些记录着特定信息的结构体，这些结构体是某些插件所需要的。该值可能会很少被用到。
+ `flags`这是一个标记变量(……不知是干什么的)。
+ `pApplicationInfo`变量存放的是`VkApplicationInfo`类型结构体的指针，该类型会在下面讲到。
+ `enabledLayerCount`加载的插件(layer)的数量，如果没有加载，则设置为0即可。
+ `ppEnabledLayerNames`加载的插件的名称的数组的指针，就是将插件列表的首地址设置到这个结构体中。
+ `enabledExtensionCount`使用的扩展(extension)的数量，如果没有使用，则设置为0即可。
+ `ppEnabledExtensionNames`同样，这里是将扩展的列表的首地址设置到这个结构体中。

### VkApplicationInfo

接下来我们看看用于提供Vulkan程序基本信息的`VkApplicationInfo`类型的定义：
```
typedef struct VkApplicationInfo {
    VkStructureType    sType;
    const void*        pNext;
    const char*        pApplicationName;
    uint32_t           applicationVersion;
    const char*        pEngineName;
    uint32_t           engineVersion;
    uint32_t           apiVersion;
} VkApplicationInfo;
```
+ `sType`和`pNext`同上面的`VkInstanceCreateInfo`中的作用相同，记录该结构体的类型以及构成结构体链。
+ `pApplicationName`, `applicationVersion`, `pEngineName`, `engineVersion`这四个属性值是可以进行自由设置，有一些工具，加载器，插件和驱动程序(tools, loaders, layers, or drivers)会通过这几个变量提供一些Debug信息或者其他报告信息，程序运行时，驱动程序可以根据这些设置进行区别化地运行。
+ `apiVersion`设置Vulkan API的版本号，暂且设置为`VK_API_VERSION_1_0`就好。

### Back to the Code

在`/* VULKAN_KEY_START */`和`/* VULKAN_KEY_END */`之间的代码，做了下面几件事情：
+ 初始化`VkApplicationInfo`结构体`app_info`和`VkInstanceCreateInfo`结构体`inst_info`；
+ 使用`vkCreateInstance()`函数创建Vulkan Instance，即`inst`；
+ 对`VkResult`类型的返回值`res`变量进行检查，如果发现创建失败则终止程序；
+ 使用`vkDestroyInstance()`对`inst`进行释放。

---

## Enumerate Devices

本章节的代码在`02-enumerate_devices.cpp`中。

 在你创建Vulkan Instance之后，Loader就会知道有多少个物理显示设备可用，而你的应用程序则需要调用Vulkan API来获取设备列表。在获取设备可用数量之后，便可以进行区别化的逻辑运算。

 ### Getting Lists of Objects from Vulkan

 获取对象列表在Vulkan操作中是非常常见的行为，Vulkan API对这种需求的策略也是一致的：返回值为**对象数量**和**对象列表指针首地址**。在使用获取对象列表的API
 时，我们需要按照下面的步骤进行：
 + 调用获取对象列表的API，传入的参数为一个整数型变量的地址和NULL指针，前者用于第一次获取对象列表的对象数量，后者告诉API本次不获取对象列表。
 + API会将对象列表的数量填到传入的整形变量地址所指向的整形变量上。
 + 应用程序需要根据API给出的对象列表数量申请足够多的内存。
 + 应用程序再次调用获取对象列表的API，传入的参数为一个整数型变量的地址，和上一步申请内存的地址，这一次API会将对象列表存储到传入的指针所指向的位置上。
该方法是使用获取对象列表API的标准流程。

### vkEnumeratePhysicalDevices

`vkEnumeratePhysicalDevices`函数，就是用来获取显示设备列表的API，在`02-enumerate_devices.cpp`文件中，我们可以看到如下使用方法：
```
uint32_t gpu_count = 1;
// Get the number of devices (GPUs) available.
VkResult U_ASSERT_ONLY res = vkEnumeratePhysicalDevices(info.inst, &gpu_count, NULL);
assert(gpu_count);
// Allocate space and get the list of devices.
info.gpus.resize(gpu_count);
res = vkEnumeratePhysicalDevices(info.inst, &gpu_count, info.gpus.data());
assert(!res && gpu_count >= 1);
```
在这里的使用方法，如上面步骤一样。
> 注意：`info.gpus` 变量的声明为 `std::vector<VkPhysicalDevice> gpus`，是一个`VkPhysicalDevice`类型的vector数组。

### The Samples_info Structure

在文件`02-enumerate_devices.cpp`中，我们看到这样的：
```
struct sample_info info = {};
init_global_layer_properties(info);
init_instance(info, "vulkansamples_enumerate");
```
在这里，所有的样例源代码文件中，会声明`sample_info`来存储一些必须的类型，并通过调用`init_instance(info, "vulkansamples_enumerate")`来简化代码，方便查看。该函数实现了上一小节中初始化的代码。`sample_info`中的`inst`会被用来放到`vkEnumeratePhysicalDevices()`使用。

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


