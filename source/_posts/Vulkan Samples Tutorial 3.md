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

**Uniform Buffer**是用来为着色器(shaders)程序提供常量数据的只读缓冲区。与深度缓冲区相同，你需要自己申请内存并填充它。

### Setting up the Uniform Data

本章样例中，我们使用Uniform Buffer将MVP(Model-View-Projection)矩阵送到着色器中，如此着色器才能对每一个顶点进行转换。

下面是创建MVP矩阵的代码：
```
info.Projection = glm::perspective(glm::radians(45.0f), 1.0f, 0.1f, 100.0f);
info.View = glm::lookAt(
    glm::vec3(-5, 3, -10), // Camera is at (-5,3,-10), in World Space
    glm::vec3(0, 0, 0),    // and looks at the origin
    glm::vec3(0, -1, 0)    // Head is up (set to 0,-1,0 to look upside-down)
    );
info.Model = glm::mat4(1.0f);
// Vulkan clip space has inverted Y and half Z.
info.Clip = glm::mat4(1.0f,  0.0f, 0.0f, 0.0f,
                      0.0f, -1.0f, 0.0f, 0.0f,
                      0.0f,  0.0f, 0.5f, 0.0f,
                      0.0f,  0.0f, 0.5f, 1.0f);

info.MVP = info.Clip * info.Projection * info.View * info.Model;
```
> 注意，`glm`库在这里用来简化代码，实际上矩阵运算需要自己写；`info.MVP`是一个4*4的矩阵。

### Creating the Uniform Buffer Object

创建Uniform缓冲区的过程与之前的深度缓冲区(Depth Buffer)基本一致，只有在`usage`属性上有所不同：
```
VkBufferCreateInfo buf_info = {};
buf_info.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
buf_info.pNext = NULL;
buf_info.usage = VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT;
buf_info.size = sizeof(info.MVP);
buf_info.queueFamilyIndexCount = 0;
buf_info.pQueueFamilyIndices = NULL;
buf_info.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
buf_info.flags = 0;
res = vkCreateBuffer(info.device, &buf_info, NULL, &info.uniform_data.buf);
assert(res == VK_SUCCESS);
```

### Allocating the Uniform Buffer Memory

与为深度缓冲区申请内存使的过程一样，先使用API计算出需要的内存大小，然后使用`memory_type_from_properties()`进行分配：
```
VkMemoryRequirements mem_reqs;
vkGetBufferMemoryRequirements(info.device, info.uniform_data.buf,
                              &mem_reqs);

VkMemoryAllocateInfo alloc_info = {};
alloc_info.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
alloc_info.pNext = NULL;
alloc_info.memoryTypeIndex = 0;

alloc_info.allocationSize = mem_reqs.size;
pass = memory_type_from_properties(info, mem_reqs.memoryTypeBits,
                                   VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT |
                                       VK_MEMORY_PROPERTY_HOST_COHERENT_BIT,
                                   &alloc_info.memoryTypeIndex);

res = vkAllocateMemory(info.device, &alloc_info, NULL,
                       &(info.uniform_data.mem));
```
使用`VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT`标记，表示分配的这块内存对CPU(Host)是可以读取的。

使用`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`标记，表示Host的内存与设备是相互可见的，并不需要刷新内存缓存。这种做法对程序是一种简化，如此便不需要调用`vkFlushMappedMemoryRanges`和`vkInvalidateMappedMemoryRanges`来确保内存的数据对GPU可见。

### Mapping and Setting the Uniform Buffer Memory

在使用深度缓冲区内存时，我们不需要去初始化它，这是因为GPU会对它的读写进行适当的处理。但在使用Uniform Buffer时，我们必须用我们要传给着色器的数据来填充它。为了让CPU能够对Uniform Buffer的内存进行读写，我们需要进行如下映射：
```
res = vkMapMemory(info.device, info.uniform_data.mem, 0, mem_reqs.size, 0,
                  (void **)&pData);
```
然后将数据复制到Uniform BUffer上，再解除映射：
```
memcpy(pData, &info.MVP, sizeof(info.MVP));
vkUnmapMemory(info.device, info.uniform_data.mem);
```
由于这块缓冲区在内存上，进行内存映射会由于页表替换等导致映射无效，因此需要立即解除映射。

最后，你只需要创建一个Buffer对象来关联内存：
```
res = vkBindBufferMemory(info.device, info.uniform_data.buf,
                         info.uniform_data.mem, 0);
```
然后，Unform Buffer就算可以使用了。

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~


































