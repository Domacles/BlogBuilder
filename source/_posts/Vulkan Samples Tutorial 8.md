---
title: Vulkan Samples Tutorial 8
date: 2017-04-28 3:00:00
tags: Vulkan
---

<!-- TOC -->

- [Draw Cube](#draw-cube)
    - [Waiting for a Swapchain Buffer](#waiting-for-a-swapchain-buffer)
    - [Beginning the Render Pass](#beginning-the-render-pass)
    - [Bind the Pipeline](#bind-the-pipeline)
    - [Bind the Descriptor Sets](#bind-the-descriptor-sets)
    - [Bind the Vertex Buffer](#bind-the-vertex-buffer)
    - [Set the Viewport and Scissors Rectangle](#set-the-viewport-and-scissors-rectangle)
    - [Draw the Vertices](#draw-the-vertices)
    - [Transitioning the Swapchain Image for Presenting](#transitioning-the-swapchain-image-for-presenting)
    - [Memory barrier approach](#memory-barrier-approach)
    - [Submit the Command Buffer](#submit-the-command-buffer)
    - [Wait for Command Buffer to Complete](#wait-for-command-buffer-to-complete)
    - [Present the Swapchain Buffer to Display](#present-the-swapchain-buffer-to-display)

<!-- /TOC -->

## Draw Cube

本章节代码在`15-draw_cube.cpp`文件中。

这里是我们需要做的最后一步，画Vlukan图像到屏幕上。

### Waiting for a Swapchain Buffer

开始画图之前，样例程序需要渲染一个目标swapchain。用函数`vkAcquireNextImageKHR()`获取swapchain列表的索引，这个函数知道用哪一个framebuffer作为渲染目标。这是可以渲染的下一个图像。
```
res = vkCreateSemaphore(info.device, &imageAcquiredSemaphoreCreateInfo,
                        NULL, &imageAcquiredSemaphore);

// Get the index of the next available swapchain image:
res = vkAcquireNextImageKHR(info.device, info.swap_chain, UINT64_MAX,
                            imageAcquiredSemaphore, VK_NULL_HANDLE,
                            &info.current_buffer);
```
第一帧，可能不需要使用信号（semaphore），因为swapchain中的所有图像都有效。但是时间证明，在用真实的命令提交给GPU开始之前，图像已准备好，我们将在后面做这些。并且如果改变样例来渲染许多帧，作为一个动画，然后在开始之前，必须去等知道用图像完成硬件。

现在，我们仅仅创建信号，并且将它和图像联系起来，以至于用这个信号推迟命令缓存的提交，直到图像准备好。

### Beginning the Render Pass

在以前的章节，我们已经定义了渲染通道，所以在命令缓存中，通过设置一个开始渲染通道来开始渲染通道的操作是简单的。
```
VkRenderPassBeginInfo rp_begin;
rp_begin.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
rp_begin.pNext = NULL;
rp_begin.renderPass = info.render_pass;
rp_begin.framebuffer = info.framebuffers[info.current_buffer];
rp_begin.renderArea.offset.x = 0;
rp_begin.renderArea.offset.y = 0;
rp_begin.renderArea.extent.width = info.width;
rp_begin.renderArea.extent.height = info.height;
rp_begin.clearValueCount = 2;
rp_begin.pClearValues = clear_values;
vkCmdBeginRenderPass(info.cmd, &rp_begin, VK_SUBPASS_CONTENTS_INLINE);
```
注意，我们已经创建一个命令缓存，并把它放入重新编码模型，通过调用`init_command_buffer()`和`execute_begin_command()`。

提供以前定义的渲染通道和从`vkAcquireNextImageKHR()`返回的索引中选择的framebuffer。

初始化这些值来设置深灰的背景颜色，并且将深度缓存的值设为“far”(`clear_values`).

`info.render_pass`中剩下的需要的信息在以前已经设置好，并且然后插入这个命令开始渲染通道到命令缓存。

### Bind the Pipeline

下一步绑定管线到命令缓存：
```
vkCmdBindPipeline(info.cmd, VK_PIPELINE_BIND_POINT_GRAPHICS, info.pipeline);
```
在前面章节中我们定义了管线并绑定，这里，告诉GPU怎样渲染后面的图元（graphics primitives）.

`VK_PIPELINE_BIND_POINT_GRAPHICS`告诉GPU这是图形管线，而不是计算管线。

注意，由于这个指令是一个命令缓存指令，所以在单个命令缓存中，程序可能定义许多图形管线，并且在他们之间转换。

### Bind the Descriptor Sets

我们已经定义的描述符集描述了着色程序怎样期望发现它的输入数据，例如MVP变换。这里是给GPU的信息：
```
vkCmdBindDescriptorSets(info.cmd, VK_PIPELINE_BIND_POINT_GRAPHICS,
                        info.pipeline_layout, 0, 1,
                        info.desc_set.data(), 0, NULL);
```
注意，在命令缓存中间，如果我们想要改变着色器程序怎样找到数据，我们可以绑定不同的描述符。例如，我们想要在命令缓存中间改变转换，我们可以用一个不同的描述符来指向一个不同的MVP转换。

### Bind the Vertex Buffer

我们在vertex_buffer样例中创建一个vertex buffer，并用顶点数据填充。这里，告诉GPU怎样找到它：
```
vkCmdBindDescriptorSets(info.cmd, VK_PIPELINE_BIND_POINT_GRAPHICS,
                        info.pipeline_layout, 0, 1,
                        info.desc_set.data(), 0, NULL);
```
这个命令绑定顶点缓存（vertex buffer）或缓存区（buffers）到命令缓存。既然这样，我们只能绑定一个缓存，但是，可以用它绑定很多。

### Set the Viewport and Scissors Rectangle

前面我们指明视口（viewport）和剪刀（scissors）是动态的状态，意味着可以用命令缓存指令（a command buffer command）设置它们。所以，我们在这里需要设置它们。下面是`init_viewports()`中设置视区的代码：
```
info.viewport.height = (float)info.height;
info.viewport.width = (float)info.width;
info.viewport.minDepth = (float)0.0f;
info.viewport.maxDepth = (float)1.0f;
info.viewport.x = 0;
info.viewport.y = 0;
vkCmdSetViewport(info.cmd, 0, NUM_VIEWPORTS, &info.viewport);
```
设置剪刀矩形（scissors rectangle）的代码类似。

使这些动态可能是最好的，因为如果在执行中窗口大小改变，许多应用需要改变这些值。这样避免了当窗口大小改变时不得不重建管线。

### Draw the Vertices

最后，

### Transitioning the Swapchain Image for Presenting

### Memory barrier approach

### Submit the Command Buffer

### Wait for Command Buffer to Complete

### Present the Swapchain Buffer to Display

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~



































