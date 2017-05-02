---
title: Vulkan Samples Tutorial 6
date: 2017-04-26 14:26:00
tags: Vulkan
---

<!-- TOC -->

- [Shaders](#shaders)
    - [Compiling GLSL Shaders into SPIR-V](#compiling-glsl-shaders-into-spir-v)
    - [Creating Vulkan Shader Modules](#creating-vulkan-shader-modules)
- [Create the Framebuffers](#create-the-framebuffers)
    - [The vulkan Framebuffer](#the-vulkan-framebuffer)
- [Create a Vertex Buffer](#create-a-vertex-buffer)
    - [Creating the Vertex Buffer Object](#creating-the-vertex-buffer-object)
    - [Allocating the Vertex Buffer Memory](#allocating-the-vertex-buffer-memory)
    - [Store the Vertex Data in the Buffer](#store-the-vertex-data-in-the-buffer)
    - [Describing the Input Vertex Data](#describing-the-input-vertex-data)
    - [Binding the Vertex Buffer to a Render Pass](#binding-the-vertex-buffer-to-a-render-pass)

<!-- /TOC -->

## Shaders

本章节代码在`11-init_shaders.cpp`文件中。

### Compiling GLSL Shaders into SPIR-V

Vulkan中底层着色器（shader）使用的代码是`SPIR-V`。样例程序调用了一个函数工具，使用着色器编译器将GLSL转换了成SPIR-V：
```
GLSLtoSPV(VK_SHADER_STAGE_VERTEX_BIT, vertShaderText, vtx_spv);
```
着色器源代码在`vertShaderText`变量中，编译SPIR-V得到的返回值在`vtx_spv`中，它是一个`unsigned int`的vector，适合于存储SPIR-V代码。

从样例代码中可以找到顶点着色器的着色源码，注意，样例同时也提供了片段着色器（fragment shader）源码，他们的编译方法和过程类似。

同时，我们应当注意到，代码中给出的着色器代码都是简单的着色器。顶点着色器只是简单地把颜色传递到输出，并且用MVP矩阵（MVP可以参考以前的章节）变换位置。片段着色器甚至更简单，仅仅传递颜色至输出。

在这个简单的样例中，有两个着色阶段：顶点着色阶段和片段着色阶段，顺序存储在`info.shaderStages`中。

### Creating Vulkan Shader Modules

在Vulkan中，通过创建`VkShaderModule`来经过编译的着色器代码，并且存储在结构体`VkPipelineShaderStageCreateInfo`中，这个结构体在后面的另一个样例中用到，作为创建整个图形管线的一部分。
```
VkShaderModuleCreateInfo moduleCreateInfo;
moduleCreateInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
moduleCreateInfo.pNext = NULL;
moduleCreateInfo.flags = 0;
moduleCreateInfo.codeSize = vtx_spv.size() * sizeof(unsigned int);
moduleCreateInfo.pCode = vtx_spv.data();
res = vkCreateShaderModule(info.device, &moduleCreateInfo, NULL,
                           &info.shaderStages[0].module);
```
注意，从GLSL转换到的SPIR-V代码是用来创建着色器模块(shader module)。用同样的过程为片段着色器创建`vkShaderModule`，并将它存储在`info.shaderStages[1].module`中。

与此同时，也可以完成一些着色器管线阶段(pipeline shader stage)中附加的creation info的初始化：
```
info.shaderStages[0].sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
info.shaderStages[0].pNext = NULL;
info.shaderStages[0].pSpecializationInfo = NULL;
info.shaderStages[0].flags = 0;
info.shaderStages[0].stage = VK_SHADER_STAGE_VERTEX_BIT;
info.shaderStages[0].pName = "main";
```
现在，着色器可以运行了。

---

## Create the Framebuffers

本章节代码在`12-init_frame_buffers.cpp`文件中。

### The vulkan Framebuffer

帧缓冲（Framebuffers）是一种被渲染通道对象使用的内存附件集合。这些内存附件的例子包括前面创建的例子中的颜色图像缓存和深度缓存。渲染时，帧缓存提供了渲染通道需要的附件。

在前面的交换链（swapchain）样例中，我们创建一个交换链，它为在交换链中的每一“帧”(frame)提供一个颜色图像附件。在深度缓存样例中，我们也创建了一个深度图象附件，它被每一帧中每一个图像附件重复使用。

在渲染通道(render pass)样例中，我们创建了一个描述附件性质的渲染通道，但是事实上我们并没有将实际资源（即images）与渲染通道连接起来。帧缓存本质上将实际的附件与渲染通道联系起来。

举个例子，如果swapchain有两帧，那么需要创建两个帧缓存。第一个帧缓存包括第一个颜色图像缓存，它可以在swapchain例子中用`vkGetSwapchainImagesKHR()`获取到。在depthbuffer例子中，第一个帧缓存也包括我们创建的单个深度缓存。第二个帧缓存包括第二个颜色图像缓存和在第一个帧缓存中使用的同一个深度缓存。

![FrameBuffers](Vulkan Samples Tutorial 6/FrameBuffers.png)

看样例代码，我们开始时声明了一个包括两个附件的数组，预初始化第二个附件给深度缓存，这是因为它会被所有的帧缓存使用。随后我们填充第一个附件。注意，"view"对象的作用是引用这个图像，由于视图对象(view)包括用来描述缓存的格式和用法的附加元数据(metadata)。
```
VkImageView attachments[2];
attachments[1] = info.depth.view;
```
接下来，填充create info结构:
```
VkFramebufferCreateInfo fb_info = {};
fb_info.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
fb_info.pNext = NULL;
fb_info.renderPass = info.render_pass;
fb_info.attachmentCount = 2;
fb_info.pAttachments = attachments;
fb_info.width = info.width;
fb_info.height = info.height;
fb_info.layers = 1;
```
样例支持运行时确定的swapchain数量，所以动态地分配帧缓存处理的数组array。
```
info.framebuffers = (VkFramebuffer *)malloc(info.swapchainImageCount *
                                            sizeof(VkFramebuffer));
```
对每一个帧缓存，设置第一个附件给swapchain中每个各自的颜色图像缓存，设置第二个附件给相同的深度缓存，所以被所有帧缓存共享。
```
for (int i = 0; i < info.swapchainImageCount; i++) {
    attachments[0] = info.buffers[i].view;
    res = vkCreateFramebuffer(info.device, &fb_info, NULL, &info.framebuffers[i]);
}
```
注意，在renderpass样例中，我们定义的相同的渲染通道也与每一个帧缓存相联系。

---

## Create a Vertex Buffer

本章节代码在`13-init_vertex_buffer.cpp`文件中。

顶点缓存（vertex buffer）是可见的CPU和可见的GPU缓存，包括描述渲染对象的几何结构的顶点数据（vertex data）。总之，顶点数据由数据的位置（x,y,z）和选择的颜色（color）、法线（normal）或其它的信息组成。像其它的3D APIs，在画图操作中，这里的方法是用顶点数据填充缓存，并把它传给GPU。

### Creating the Vertex Buffer Object

创建vertex buffer与创建uniform buffer几乎一致，就像在uniform sample中一样，以创建一个缓存对象开始。
```
VkBufferCreateInfo buf_info = {};
buf_info.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
buf_info.pNext = NULL;
buf_info.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
buf_info.size = sizeof(g_vb_solid_face_colors_Data);
buf_info.queueFamilyIndexCount = 0;
buf_info.pQueueFamilyIndices = NULL;
buf_info.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
buf_info.flags = 0;
res = vkCreateBuffer(info.device, &buf_info, NULL, &info.vertex_buffer.buf);
```
创建uniform buffer对象与vertex buffer对象实际上唯一不同是`usage`参数的设置。

立方体数据（`g_vb_solid_face_colors_Data`）由定义了12个三角形的36个顶点构成，立方体的6个面中，每一个面包括2个三角形。每一个三角形都有一个面的颜色。从`cube_data.h`中可以看到真实的数据。

### Allocating the Vertex Buffer Memory

此外，分配vertex buffer的步骤与分配uniform buffer的步骤相似。首先，查询获取内存的requirements，它包括考虑机器的限制条件，如对齐（alignment）。从样例中的代码中可以看到这个过程与uniform样例非常相似。

### Store the Vertex Data in the Buffer

一旦vertex buffer的内存分配完，然后进行内存映射操作、使用顶点数据进行初始化、之后进行解除映射操作(在之前使用uniform buffer时，解释过为何要在使用之后立即解除映射)：
```
uint8_t *pData;
res = vkMapMemory(info.device, info.vertex_buffer.mem, 0,
                  mem_reqs.size, 0, (void **)&pData);

memcpy(pData, g_vb_solid_face_colors_Data,
       sizeof(g_vb_solid_face_colors_Data));

vkUnmapMemory(info.device, info.vertex_buffer.mem);
```
最后一步，将内存绑定到缓存对象上。
```
res = vkBindBufferMemory(info.device, info.vertex_buffer.buf,
                         info.vertex_buffer.mem, 0);
```

### Describing the Input Vertex Data

样例中的顶点数据如下所示：
```
struct Vertex {
    float posX, posY, posZ, posW; // Position data
    float r, g, b, a;             // Color
};
```
我们需要创建一个顶点输入的绑定来为GPU描述数据的组织形式。虽然是在这里进行`vi_binding`和`vi_attribs`成员的设置，但他们将在后面的样例中作为创建图形管线的一部分。注意看西面顶点数据的格式，这是个很好的参考样例：
```
info.vi_binding.binding = 0;
info.vi_binding.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;
info.vi_binding.stride = sizeof(g_vb_solid_face_colors_Data[0]);

info.vi_attribs[0].binding = 0;
info.vi_attribs[0].location = 0;
info.vi_attribs[0].format = VK_FORMAT_R32G32B32A32_SFLOAT;
info.vi_attribs[0].offset = 0;
info.vi_attribs[1].binding = 0;
info.vi_attribs[1].location = 1;
info.vi_attribs[1].format = VK_FORMAT_R32G32B32A32_SFLOAT;
info.vi_attribs[1].offset = 16;
```
`stride`表示一个顶点数据的大小，也可以表示成获取下一个顶点数据需要的字节长度。

`binding`和`location`成员表示他们在GLSL着色器源码中各自的值。可以在shader样例中回顾shader源码。

尽管我们让`vi_attribs[0]`表示顶点位置数据的信息，但在`info.vi_attribs[0]`中，实际上是`format`属性用4个字节颜色格式(即`VK_FORMAT_R32G32B32A32_SFLOAT`)来为GPU描述数据格式的。那`vi_attribs[1]`很自然地使用来表示颜色数据，所以我们可以看到`info.vi_attribs[1].format`的颜色格式设置。

`offset`成员表示顶点数据中每个属性的偏移量(在OpenGL中也有这个概念，即第一个顶点数据是从0字节开始的，第一个颜色数据是从16字节开始的，然后根据它们的`format`来计算下一个数据的位置)。

### Binding the Vertex Buffer to a Render Pass

由于后面样例可以看到，我们可以掠过render pass中的大部分代码。然后找到绑定vertex buffer到render pass的代码：
```
vkCmdBeginRenderPass(info.cmd, &rp_begin, VK_SUBPASS_CONTENTS_INLINE);

vkCmdBindVertexBuffers(info.cmd, 0,             /* Start Binding */
                       1,                       /* Binding Count */
                       &info.vertex_buffer.buf, /* pBuffers */
                       offsets);                /* pOffsets */

vkCmdEndRenderPass(info.cmd);
```
注意，在渲染通道中，只能用一个render pass链接vertex buffer；也就是，当记录命令缓冲区时，在`vkCmdBeginRenderPass`和`vkCmdEndRenderPass`之间的代码进行额。当画图时，必须告诉GPU用什么顶点缓存。

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~



































