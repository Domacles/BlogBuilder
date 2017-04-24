---
title: Vulkan Samples Tutorial 4
date: 2017-04-24 11:00:00
tags: Vulkan
---

<!-- TOC -->

- [Descriptor Set Layouts and Pipeline Layouts](#descriptor-set-layouts-and-pipeline-layouts)
    - [Descriptors and Descriptor Sets](#descriptors-and-descriptor-sets)
    - [Descriptor Set Layouts](#descriptor-set-layouts)
    - [Pipeline Layouts](#pipeline-layouts)
    - [Shader Referencing of Descriptors](#shader-referencing-of-descriptors)
- [Create a Descriptor Set](#create-a-descriptor-set)
    - [Descriptor Pool](#descriptor-pool)
    - [Allocate a Descriptor Set from the Pool](#allocate-a-descriptor-set-from-the-pool)
    - [Update the Descriptor Set](#update-the-descriptor-set)

<!-- /TOC -->

## Descriptor Set Layouts and Pipeline Layouts

本章节代码在`08-init_pipeline_layout.cpp`中。

在之前的样例中，我们创建了Uniform Buffer，但我们还没有让shader来使用它。我们使用**Descriptor**来完成这一工作。

### Descriptors and Descriptor Sets

本段讲解什么是Descriptor，Descriptor是一种不透明的特殊着色器变量，作为一种间接的方式，着色器用它来存取缓冲区数据和图像资源。我们可以认为它是资源的指针(pointer)。Vulkan API中，支持在绘制操作过程中更改这些变量，如此着色器才能够为每次绘制使用不同的资源。

本样例作为一个简单的样例，我们只用了1个Uniform Buffer，但也有可能我们会创建两个Uniform Buffer，每一个都存放着不同的MVP矩阵，从而能够绘制不同场景。如果我们有两个Uniform Buffer，我们可以很轻松地改变Descriptor的指向，来切换MVP矩阵的使用。

Descriptor Set表示：该集合中的Descriptor指向的是同类型的资源，这些资源是使用相同的**Layout Binding**。

在本样例中我们没有用到**材质**(texture)，但可能我们会创建一个由两个Descriptor组成的集合，每个Descriptor引用着不同的材质，通过使用这个Descriptor Set，让这两种材质在绘制的过程中始终可用。命令缓冲区中存储的命令可以通过指定材质的索引来选择材质来使用。

> 注意，在本章节这里，我们并没有实际创建分类Descriptor Set，我们在本章节主要做的是“描述(describing)”Descriptor Set的配置属性。

为了能够设置Descriptor Set，我们需要使用Descriptor Set Layout。

### Descriptor Set Layouts

Descriptor Set Layout用于设置集合中的内容。我们需要为每一个Descriptor集合绑定一个layout，就像下面这段代码：
```
VkDescriptorSetLayoutBinding layout_binding = {};
layout_binding.binding = 0;
layout_binding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
layout_binding.descriptorCount = 1;
layout_binding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
layout_binding.pImmutableSamplers = NULL;
```
上面的代码我们实现了下面四个方面：
+ 创建了一个Descriptor集合，并绑定到了0这个对象上，即将`binding`设置为0。
+ 该Descriptor集合指向的是一个Uniform Buffer，即将`descriptorType`指定为我们需要的类型。
+ 在这个Descriptor集合中，只有一个Descriptor对象，数量在`descriptorCount`上设置的。
+ 最后，`stageFlags`指明了该Descriptor集合在顶点着色器执行阶段有效。

在绑定Descriptor对象之后，紧接着我们需要创建Descriptor Set Layout对象：
```
#define NUM_DESCRIPTOR_SETS 1
VkDescriptorSetLayoutCreateInfo descriptor_layout = {};
descriptor_layout.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
descriptor_layout.pNext = NULL;
descriptor_layout.bindingCount = 1;
descriptor_layout.pBindings = &layout_binding;
info.desc_layout.resize(NUM_DESCRIPTOR_SETS);
res = vkCreateDescriptorSetLayout(info.device, &descriptor_layout, NULL,
                                  info.desc_layout.data());
```

### Pipeline Layouts

Pipeline Layout包含了一系列的Descriptor Set Layout。在定义Descriptor Set之后，我们就要完成Pipeline Layout的CreateInfo的填充了，在创建Pipeline Layout之后，就会创建我们之前定义的Descriptor了。

创建Pipeline Layout的代码：
```
VkPipelineLayoutCreateInfo pPipelineLayoutCreateInfo = {};
pPipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pPipelineLayoutCreateInfo.pNext = NULL;
pPipelineLayoutCreateInfo.pushConstantRangeCount = 0;
pPipelineLayoutCreateInfo.pPushConstantRanges = NULL;
pPipelineLayoutCreateInfo.setLayoutCount = NUM_DESCRIPTOR_SETS;
pPipelineLayoutCreateInfo.pSetLayouts = info.desc_layout.data();

res = vkCreatePipelineLayout(info.device, &pPipelineLayoutCreateInfo, NULL,
                             &info.pipeline_layout);
```
完成上面的工作后，在接下来章节的内容，我们就可以用Pipeline Layout来创建**图形管线**（graphics pipeline）了。

### Shader Referencing of Descriptors

在着色器语言中，明确引用Descriptors是很有用的一个做法。

举个例子，在GLSL中：
```
 layout (set=M, binding=N) uniform sampler2D variableNameArray[I];
```
这一句告诉我们：
+ M表示`pSetLayouts`管线成员中的第M个Descriptor Set Layout。
+ N表示M的`pBingings`的第N个Descriptor Set。
+ I表示Descriptor Set数组化的下标。

在顶点着色器中使用Uniform Buffer的Layout的代码如下：
```
layout (std140, binding = 0) uniform bufferVals {
    mat4 mvp;
} myBufferVals;
```
这行代码表示Uniform Buffer中的内容映射到了`myBufferVals`结构体中；`set=M`没有指定，这里会默认M为0。`std140`是一个Uniform数据块打包的协议标准，你可以引用它，来将更多的数据塞到一个Uniform数据块中。如果你希望得到关于这个标准更多的信息，则点击[这里](https://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt)来获取更多的信息。

---

## Create a Descriptor Set

本章节的代码在`09-init_descriptor_set.cpp`文件中。

回想一下上一章的内容，我们定义了descriptor set layout，但并没有立即创建descriptor set。descriptor set是用于告诉GPU如何组织Uniform BUffer的数据的。现在，我们就需要开始创建和初始化descriptor set。

### Descriptor Pool

就像命令缓冲区(Command Buffer)一样，descriptor set是从缓冲池(pool)中分配的。所以，我们第一步就是创建这个**Descriptor Pool**。因为我们样例只需要为一个uniform buffer创建一个descriptor set，可以使用下面的代码直接来搞：
```
VkDescriptorPoolSize type_count[1];
type_count[0].type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
type_count[0].descriptorCount = 1;

VkDescriptorPoolCreateInfo descriptor_pool = {};
descriptor_pool.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
descriptor_pool.pNext = NULL;
descriptor_pool.maxSets = 1;
descriptor_pool.poolSizeCount = 1;
descriptor_pool.pPoolSizes = type_count;

res = vkCreateDescriptorPool(info.device, &descriptor_pool, NULL,
                             &info.desc_pool);
```

### Allocate a Descriptor Set from the Pool

然后，我们需要从缓冲池中创建一个descriptor set。注意，这里我们需要提供在之前章节创建的descriptor set layout，这个layout描述了该descriptor set应当如何来创建。

下面是从缓冲池中创建descriptor set的代码：
```
VkDescriptorSetAllocateInfo alloc_info[1];
alloc_info[0].sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
alloc_info[0].pNext = NULL;
alloc_info[0].descriptorPool = info.desc_pool;
alloc_info[0].descriptorSetCount = NUM_DESCRIPTOR_SETS;
alloc_info[0].pSetLayouts = info.desc_layout.data();
info.desc_set.resize(NUM_DESCRIPTOR_SETS);
res = vkAllocateDescriptorSets(info.device, alloc_info, info.desc_set.data());
```

### Update the Descriptor Set

~~注意，事实上，我们还不能够随时随地使用uniform buffer的句柄来操作其中的数据。~~

在函数`init_uniform_buffer()`中，我们可以看到uniform buffer信息被存储到一个叫做`info.uniform_data.buffer_info`的VkDescriptorBufferInfo结构体对象中，该结构体对象在函数中被初始化。

`info.uniform_data.buffer_info`的结构体：
```
typedef struct VkDescriptorBufferInfo {
    VkBuffer        buffer;
    VkDeviceSize    offset;
    VkDeviceSize    range;
} VkDescriptorBufferInfo;
```
`buffer`成员包含着uniform buffer的句柄。

```
VkWriteDescriptorSet writes[1];
writes[0] = {};
writes[0].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
writes[0].pNext = NULL;
writes[0].dstSet = info.desc_set[0];
writes[0].descriptorCount = 1;
writes[0].descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
writes[0].pBufferInfo = &info.uniform_data.buffer_info;
writes[0].dstArrayElement = 0;
writes[0].dstBinding = 0;

vkUpdateDescriptorSets(info.device, 1, writes, 0, NULL);
```
上面的步骤，实质上是将`VkDescriptorBufferInfo`复制到descriptor上，如同device内存一样。

info的buffer——info包含着uniform buffer的句柄，也有uniform buffer中数据的地址偏移(offset)和大小(size)。在本样例中，因为uniform buffer只包含这MVP转换矩阵，所以这里uniform data的数据偏移为0，数据大小为MVP矩阵的大小。

descriptor的二进制实现细节可能对于不同设备有所不同，且对于我们是不透明的。这就是为什么我们要使用Vulkan驱动中的函数来操作descriptor对象，而不是自己直接通过内存映射和内存读写的原因。

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~
























