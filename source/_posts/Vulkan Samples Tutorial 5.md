---
title: Vulkan Samples Tutorial 5
date: 2017-04-25 00:00:00
tags: Vulkan
---

<!-- TOC -->

- [Create a Render Pass](#create-a-render-pass)
    - [Image Layout Transition](#image-layout-transition)
    - [Create the Render Pass](#create-the-render-pass)

<!-- /TOC -->

## Create a Render Pass

本章节代码在`10-init_render_pass.cpp`文件中。

**渲染层(Render Pass)**指的是：指定的附件集合(collection of attachments)、子层(subpasses)、渲染过程用到的依赖(dependencies)这一些概念的渲染操作范畴。1个渲染层至少包含1个子层。传递给驱动的信息，可以使驱动了解程序希望什么时候开始渲染以及渲染什么东西，还可以让驱动提供渲染操作的硬件优化急速。

我们通过使用`vkCreateRenderPass()`来定义渲染层，然后使用`vkCmdBeginRenderPass()`和`vkCmdEndRenderPass()`向命令缓冲区插入一个渲染层实例。

在本章节内容中，我们只是定义并创建渲染层，但直到本章节代码结束都没有将它插入到命令缓冲区中。

在本章节样例中，渲染层附件(render pass attachments)用到了颜色附件(color attachment)，就是从Swapchian获得的图像，还用到了深度/模版附件(depth/stencil attachment)，附件使用的深度缓冲区是在之前创建的。

图像附件必须在使用时准备好，这样它们才能够被添加到渲染层实例上用于在命令缓冲区中执行。准备的过程包括图像布局(Image Layouts)从初始未定义阶段转换到在渲染层优化阶段的过程。我们将会在本章节了解在创建渲染层之前，图像布局的转换过程。

### Image Layout Transition

The Need for Alternate Memory Access Patterns

图像的布局(Image Layouts)指的是图像像素如何从二维坐标系映射到图像内存偏移的方法。比较有代表性的布局是，图像数据使用的线性映射：2D图像按照一行接着一行的方法存储到连续的内存中：
```
offset = rowCoord * pitch + colCoord
```
`pitch`表示一行的大小。该变量通常是和图像的宽度相同，但通常会填充一些字节，使得图像数据的起始位置满足GPU内存地址的对齐要求。

线性的布局很适合沿着行来读写连续的像素，即按照`colCoord`变化的方向，但是，多数图形操作包含一些读取相邻行像素的操作，即按照`rowCoord`的方向。如果图像非常宽，那采用线性映射的方法会导致每次存取相邻行内存地址的跨度很大。在多级缓存内存系统中，由于内存地址转换跨度过大造成TLB未命中或者缓存未命中太多，导致性能低下问题的出现。

为了减少这些低效现象的繁盛，许多GPU硬件实现支持优化optimal/tiled的内存存取方案。在一种优化的布局设计中，图像中间的矩形形状的像素被存储到一段连续的内存中，其他的矩形形状的像素也按照中心矩阵的存放方法存到内存中。举个粒子，由左上角[16, 32]到右下角[31, 47]建立的像素矩阵，可以圈出一块16 x 16的像素块，该像素块存到一块连续的内存上。行与行之间的间隔就变得不长了。

如果GPU想要填充上面所讲的像素块，比如可以使用先沟通的颜色，这种方法就可以用一个相对开销较小的内存操作方法来填充这256个像素组成的像素块。

下面是一个简单的2 x 2的平铺图，注意，蓝色的像素在线性内存布局中两部分想个比较远，而在平铺布局中相隔就比较近：

![ImageMemoryLayout](Vulkan Samples Tutorial 5/ImageMemoryLayout.png)

大部分图像内存布局的实现，采用更复杂的平铺方法，且平铺的大小大于2 x 2(这让我想起了用**希尔伯特曲线**填充平面空间的方法)。

Vulkan Control Over the Layout

由于上面所解释的那样，GPU会为了提升渲染效率才支持优化的布局方案。优化的布局方案通常是**不透明的**(opaque)，这表示优化布局方案格式的具体细节并不是公开的或者不会让别人知道如何读写图像数据。

举个例子，假如你想让GPU使用一种优化的布局方案来渲染图像，但是如果你希望让CPU读取和理解最终需要渲染的图像的数据，你需要从优化的布局方案改成一般的布局方案。

在Vulkan中，从一种布局转换到另外一种布局叫做**布局转换**(layout transition)。下面三种途径的任何一种，都能够让我们实现布局转换：
1. 内存栅格化命令(Memory Barrier Command)，通过`vkCmdPipelineBarrier`来控制
2. 了解渲染层最终布局规范(Render Pass final layout specification)
3. 了解渲染层子层布局规范(Render Pass subpass layout specification)

内存栅格化命令(Memory Barrier Command)，是一个可以在命令缓冲区中显式调用的布局转换命令。应用举例，比如我们可以通过这个命令来同步多个复杂场景的内存访问。上面的方法一旦采用另外两种方法来实现布局转换，那就不能用内存栅格化命令来实现布局转换了。

通常，在渲染开始之前和渲染完成之后，需要进行图形布局转换以适应图像渲染。渲染开始前的第一次转换，会为GPU准备好适合的图像格式用于渲染；渲染完成后的最后一次转换，用于将图像呈现给显示设备。在这些情况下，我们需要将布局转换方式指定为渲染层定义的一部分。在本章节我们将看到是如何具体做的。

一条布局转换命令，可能会触发也可能不会触发事实上GPU布局转换操作。比如，如果旧的布局格式未定义，且新的布局格式是优化的，那GPU就得什么都不做，除非程序已经安排GPU硬件采用优化的模式访问内存。这是因为图像内容处于未定义状态下，不能够通过转换它的格式来进行另存储。另外，如果旧的数据布局格式是一般的，朴素的，且有迹象表明需要图像数据进行另保存，那转换成优化布局时，可能会涉及到GPU的一些重新调整像素位置的工作，即GPU会进行布局转换操作。

即使你知道或者认为一条布局转换操作不会实际发生，但最好的实践方式还是在需要的地方进行添加布局转换操作，这是因为这样可以给驱动程序更多的信息，来帮助驱动软件确保你的应用程序能够在更多平台上执行正确。

Image Layout Transitions in the Samples

下面的样例采用的子层定义(subpass definitions)和渲染层定义(render pass definitions)来指定需要的图像格式的布局转换，而不是采用内存栅格化命令。

![RenderPassLayouts](Vulkan Samples Tutorial 5/RenderPassLayouts.png)

初始的渲染层布局时未定义的，在这里，我们是告诉了驱动渲染层最初的布局格式是未定义的，驱动就不会进行任何转换直到子层渲染开始或者渲染层结束。(这段不知道说的是什么意思…………)

子层布局格式被设置成对颜色缓冲区的优化布局，这就告诉驱动子层渲染过程中，需要对子层布局格式进行转换成优化的布局格式。对深度缓冲区的优化布局的设置过程是相似的。

最终渲染层布局会告诉驱动要将布局转换成最优的布局格式用来显示。

### Create the Render Pass



Attachments

Subpass

Render Pass

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~



































