---
title: Vulkan Samples Tutorial 3
date: 2017-04-11 00:12:13
tags: Vulkan
---

<!-- TOC -->

- [Depth Buffer](#depth-buffer)
    - [Create a Depth Buffer](#create-a-depth-buffer)
    - [Create the Depth Buffer Image Object](#create-the-depth-buffer-image-object)
    - [Allocate the Memory for the Depth Buffer](#allocate-the-memory-for-the-depth-buffer)
    - [Bind the Memory to the Depth Buffer](#bind-the-memory-to-the-depth-buffer)
    - [Create the Image View](#create-the-image-view)
- [Uniform Buffer](#uniform-buffer)
    - [Create a Uniform Buffer](#create-a-uniform-buffer)
    - [Setting up the Uniform Data](#setting-up-the-uniform-data)
    - [Creating the Uniform Buffer Object](#creating-the-uniform-buffer-object)
    - [Allocating the Uniform Buffer Memory](#allocating-the-uniform-buffer-memory)
    - [Mapping and Setting the Uniform Buffer Memory](#mapping-and-setting-the-uniform-buffer-memory)

<!-- /TOC -->

Depth Buffer和Uniform Buffer在OpenGL也有对应的概念，可以参考OpenGL的讲述来理解这两个缓冲区的作用。

## Depth Buffer

本章节代码在`06-init_depth_buffer.cpp`文件中。

### Create a Depth Buffer

深度缓冲区（Depth Buffer）可以可选的，在最后的样例工程中要渲染3D立方体，就必须使用深度缓冲区。深度缓冲区对于每个Swapchain的Image都是可以复用的，因此，深度缓冲区应当只使用一个，即便Swapchain中有很多image对象。

不同于`vkCreateSwapchainKHR()`，当Swapchain的每个Image对象创建时，你需要自己去创建和分配深度缓冲区给Image：

1. 创建使用深度缓冲区Image对象
2. 深度缓冲区所在**设备**上分配内存
3. 把申请的内存绑定到Image对象上
4. 创建使用深度缓冲区的Image View

以上步骤可以使用下图表示：
![DepthBufferBindView](Vulkan Samples Tutorial 3/DepthBufferBindView.png)

### Create the Depth Buffer Image Object

与之前一些使用`CreateInfo`创建对象的过程相同，创建使用深度缓冲区的Image对象的代码如下：
```
mage_info.sType = VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO;
image_info.pNext = NULL;
image_info.imageType = VK_IMAGE_TYPE_2D;
image_info.format = VK_FORMAT_D16_UNORM;
image_info.extent.width = info.width;
image_info.extent.height = info.height;
image_info.extent.depth = 1;
image_info.mipLevels = 1;
image_info.arrayLayers = 1;
image_info.samples = NUM_SAMPLES;
image_info.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
image_info.usage = VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT;
image_info.queueFamilyIndexCount = 0;
image_info.pQueueFamilyIndices = NULL;
image_info.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
image_info.flags = 0;

vkCreateImage(info.device, &image_info, NULL, &info.depth.image);
```
上面这份代码仅仅为Image创建了一个对象，还并没有为它申请深度缓冲区内存。但在创建时，你需要设置足够的信息用于创建匹配窗口大小的深度缓冲区。

### Allocate the Memory for the Depth Buffer

在申请分配深度缓冲区内存之前，我们需要知道要申请的内存大小。但由于GPU硬件上的内存有对齐约束，而这些约束不同设备之间可能有所不同，因此，我们需要通过Vulkan的API来查询我们需要的内存大小、对齐方式、内存类型等信息：
```
vkGetImageMemoryRequirements(info.device, info.depth.image, &mem_reqs);
```
然后我们需要用获取到的`VkMemoryRequirements`对象，来填充`VkMemoryAllocateInfo`这个Info对象：
```
VkMemoryAllocateInfo mem_alloc = {};
mem_alloc.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
mem_alloc.pNext = NULL;
mem_alloc.allocationSize = mem_reqs.size;
pass = memory_type_from_properties(info, mem_reqs.memoryTypeBits,
        0, /* No Requirements */
        &mem_alloc.memoryTypeIndex);
```
`memory_type_from_properties()`是一个用于确定适合的内存指针类型的函数。最后，使用下面的代码来进行分配内存：
```
vkAllocateMemory(info.device, &mem_alloc, NULL, &info.depth.mem);
```

### Bind the Memory to the Depth Buffer

在拥有一个Image对象且申请到了设备上的深度缓冲区内存之后，就可以将缓冲区内存帮到Image对象上：
```
vkBindImageMemory(info.device, info.depth.image, info.depth.mem, 0);
```

### Create the Image View

接下来就是按照自己的需要，设置ViewCreateInfo对象，然后与之前相同的流程，使用`vkCreateImageView()`函数创建对象：
```
VkImageViewCreateInfo view_info = {};
view_info.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
view_info.pNext = NULL;
view_info.image = info.depth.image;
view_info.format = VK_FORMAT_D16_UNORM;
view_info.components.r = VK_COMPONENT_SWIZZLE_R;
view_info.components.g = VK_COMPONENT_SWIZZLE_G;
view_info.components.b = VK_COMPONENT_SWIZZLE_B;
view_info.components.a = VK_COMPONENT_SWIZZLE_A;
view_info.subresourceRange.aspectMask = VK_IMAGE_ASPECT_DEPTH_BIT;
view_info.subresourceRange.baseMipLevel = 0;
view_info.subresourceRange.levelCount = 1;
view_info.subresourceRange.baseArrayLayer = 0;
view_info.subresourceRange.layerCount = 1;
view_info.viewType = VK_IMAGE_VIEW_TYPE_2D;
view_info.flags = 0;

res = vkCreateImageView(info.device, &view_info, NULL, &info.depth.view);
```
---

## Uniform Buffer

本章节代码在`07-init_uniform_buffer.cpp`文件中。

### Create a Uniform Buffer

### Setting up the Uniform Data

### Creating the Uniform Buffer Object

### Allocating the Uniform Buffer Memory

### Mapping and Setting the Uniform Buffer Memory

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~


































