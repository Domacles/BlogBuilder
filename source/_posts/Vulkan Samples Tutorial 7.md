---
title: Vulkan Samples Tutorial 7
date: 2017-04-27 3:20:00
tags: Vulkan
---

<!-- TOC -->

- [Create a Graphics Pipeline](#Create-a-Graphics-Pipeline)
    - [Dynamic State](#Dynamic-State)
    - [Pipeline Vertex Input State](#Pipeline-Vertex-Input-States)
    - [Pipeline Vertex Input Assembly State](#Pipeline-Vertex-Input-Assembly-State)
    - [Pipeline Rasterization State](#Pipeline-Rasterization-State)
    - [Pipeline Color Blend State](#Pipeline-Color-Blend-State)
    - [Pipeline Viewport State](#Pipeline-Viewport-State)
    - [Pipeline Depth Stencil State](#Pipeline-Depth-Stencil-State)
    - [Pipeline Multisample State](#Pipeline-Multisample-State)
    - [Pulling It All Together - Create Graphics Pipeline](#Pulling-It-All-Together---Create-Graphics-Pipeline)

<!-- /TOC -->

## Create a Graphics Pipeline

本章节代码在`14-init_pipeline.cpp`文件中。

我们将更近地将所有放到一起去画立方体。下一步通过设置图形管线（graphics pipeline）来设置渲染的GPU。

图形管线由着色器阶段、管线布局、渲染通道和固定功能的管线阶段。在以前的章节中定义了着色器阶段和管线布局。在此，将设置余下的固定功能的管线阶段。包括填充一些创建管线的"create info"数据结构。在将片段放到片段缓存之前，这里完成的大部分工作是设置预片段操作。简图如下：

![GraphicsPipeline.](Vulkan Samples Tutorial/GraphicsPipeline..png)

下一步将设置管线状态对象，即上图中右下角的灰色框。最后一步是连接其它对象指向左上角的紫色管线框，为了完成图形管线的定义。

### Dynamic State

在命令缓冲区运行期间，动态管线状态是一种能被命令缓冲区改变的状态。什么状态是动态的预先通知可能对驱动器有用，因为它设置命令缓存执行的GPU。

样例提供了状态列表，在命令缓存执行中改变状态。在此，代码以设置动态状态列表开始，开始时他们的状态都是disabled。
```
VkDynamicState dynamicStateEnables[VK_DYNAMIC_STATE_RANGE_SIZE];
VkPipelineDynamicStateCreateInfo dynamicState = {};
memset(dynamicStateEnables, 0, sizeof dynamicStateEnables);
dynamicState.sType = VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
dynamicState.pNext = NULL;
dynamicState.pDynamicStates = dynamicStateEnables;
dynamicState.dynamicStateCount = 0;
```
样例表明用命令缓存动态地改变一些状态，所以，当设置视区（viewport）和剪刀（scissors）矩形时，改变`dynamicStateEnables`数组。代码通过修改`dynamicStateEnables`来保持清楚地设置viewport和scissors。

### Pipeline Vertex Input State

当创建vertex buffer时，已经对vertex输入状态进行了初始化，因为它在此时的设置很简单。输入状态包括顶点数据的格式和管理。可以回顾vertexbuffer样例去查看`vi_binding`和`vi_attribs`变量是如何设置的。
```
VkPipelineVertexInputStateCreateInfo vi;
vi.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vi.pNext = NULL;
vi.flags = 0;
vi.vertexBindingDescriptionCount = 1;
vi.pVertexBindingDescriptions = &info.vi_binding;
vi.vertexAttributeDescriptionCount = 2;
vi.pVertexAttributeDescriptions = info.vi_attribs;
```

### Pipeline Vertex Input Assembly State

输入组件的状态（input assembly state）主要是什么时候声明顶点如何从我们想要画的几何图形中得到。例如，顶点可能形成三角带（triangle strip）和三角扇（triangle fan）。这里，我们仅仅用三角形列表，每三个点描述一个三角形。
```
VkPipelineInputAssemblyStateCreateInfo ia;
ia.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
ia.pNext = NULL;
ia.flags = 0;
ia.primitiveRestartEnable = VK_FALSE;
ia.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
```

### Pipeline Rasterization State

在GPU中，下面的数据结构设置了网格化操作。
```
VkPipelineRasterizationStateCreateInfo rs;
rs.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
rs.pNext = NULL;
rs.flags = 0;
rs.polygonMode = VK_POLYGON_MODE_FILL;
rs.cullMode = VK_CULL_MODE_BACK_BIT;
rs.frontFace = VK_FRONT_FACE_CLOCKWISE;
rs.depthClampEnable = VK_TRUE;
rs.rasterizerDiscardEnable = VK_FALSE;
rs.depthBiasEnable = VK_FALSE;
rs.depthBiasConstantFactor = 0;
rs.depthBiasClamp = 0;
rs.depthBiasSlopeFactor = 0;
rs.lineWidth = 1.0f;
```
用常见值设置这些字段。我们可以识别`frontFace`成员和GL函数`glFrontFace()`的相互关系。

### Pipeline Color Blend State

绑定是另一个`end of the fixed pipe`操作，即在目的地设置像素的简单替换。
```
VkPipelineColorBlendStateCreateInfo cb;
cb.sType = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
cb.pNext = NULL;
cb.flags = 0;
VkPipelineColorBlendAttachmentState att_state[1];
att_state[0].colorWriteMask = 0xf;
att_state[0].blendEnable = VK_FALSE;
att_state[0].alphaBlendOp = VK_BLEND_OP_ADD;
att_state[0].colorBlendOp = VK_BLEND_OP_ADD;
att_state[0].srcColorBlendFactor = VK_BLEND_FACTOR_ZERO;
att_state[0].dstColorBlendFactor = VK_BLEND_FACTOR_ZERO;
att_state[0].srcAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
att_state[0].dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
cb.attachmentCount = 1;
cb.pAttachments = att_state;
cb.logicOpEnable = VK_FALSE;
cb.logicOp = VK_LOGIC_OP_NO_OP;
cb.blendConstants[0] = 1.0f;
cb.blendConstants[1] = 1.0f;
cb.blendConstants[2] = 1.0f;
cb.blendConstants[3] = 1.0f;
```
在每个附件基础上，都提供了一些配置信息的。在管线中，每一个颜色附件需要一个`VkPipelineColorBlendAttachmentState`。既然这样，只有一个颜色附件。

`colorWriteMask`从R, G, B和支持写（writing）的组件（components）。这里，可以选择这4个组件。

禁用`blendEnable`意味着与绑定有点的`att_state[0]`中余下的设置没关系。

我们也可以禁用pixel-writing逻辑操作，因为，当写像素到帧缓存时，这个样例仅仅做了一个简单的替换。

绑定常量用于一些"blend factors"(如，`VK_BLEND_FACTOR_CONSTANT_COLOR`)，并且仅仅设置合理的内容，同样地在这个样例没有使用。

### Pipeline Viewport State

draw_cube样例在命令缓存中用命令设置viewport和scissors矩形。这个代码告诉驱动那些viewport和scissors状态是动态的，并且忽略`pViewPorts`和`pScissors`。既然这样，只有一个颜色附件。
```
VkPipelineViewportStateCreateInfo vp = {};
vp.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
vp.pNext = NULL;
vp.flags = 0;
vp.viewportCount = 1;
dynamicStateEnables[dynamicState.dynamicStateCount++] = VK_DYNAMIC_STATE_VIEWPORT;
vp.scissorCount = 1;
dynamicStateEnables[dynamicState.dynamicStateCount++] = VK_DYNAMIC_STATE_SCISSOR;
vp.pScissors = NULL;
vp.pViewports = NULL;
```

### Pipeline Depth Stencil State

继续通过设置深度缓存来初始化后端固定功能。
```
VkPipelineDepthStencilStateCreateInfo ds;
ds.sType = VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO;
ds.pNext = NULL;
ds.flags = 0;
ds.depthTestEnable = VK_TRUE;
ds.depthWriteEnable = VK_TRUE;
ds.depthCompareOp = VK_COMPARE_OP_LESS_OR_EQUAL;
ds.depthBoundsTestEnable = VK_FALSE;
ds.minDepthBounds = 0;
ds.maxDepthBounds = 0;
ds.stencilTestEnable = VK_FALSE;
ds.back.failOp = VK_STENCIL_OP_KEEP;
ds.back.passOp = VK_STENCIL_OP_KEEP;
ds.back.compareOp = VK_COMPARE_OP_ALWAYS;
ds.back.compareMask = 0;
ds.back.reference = 0;
ds.back.depthFailOp = VK_STENCIL_OP_KEEP;
ds.back.writeMask = 0;
ds.front = ds.back;
```
因为我们想深度缓存，所以深度缓存writing和testing有效。此外，设置深度缓存操作与`VK_COMPARE_OP_LESS_OR_EQUAL`相比较。最后，disable掉模板（stencil）操作，因为这个样例不需要它。

### Pipeline Multisample State

这个例子我们不打算做任何精细的多采样，所以通过建立无多采样来完成管线设置。
```
VkPipelineMultisampleStateCreateInfo ms;
ms.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
ms.pNext = NULL;
ms.flags = 0;
ms.pSampleMask = NULL;
ms.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;
ms.sampleShadingEnable = VK_FALSE;
ms.alphaToCoverageEnable = VK_FALSE;
ms.alphaToOneEnable = VK_FALSE;
ms.minSampleShading = 0.0;
```

### Pulling It All Together - Create Graphics Pipeline

最后，我们综合所有需要的信息来创建管线。
```
VkGraphicsPipelineCreateInfo pipeline;
pipeline.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipeline.pNext = NULL;
pipeline.layout = info.pipeline_layout;
pipeline.basePipelineHandle = VK_NULL_HANDLE;
pipeline.basePipelineIndex = 0;
pipeline.flags = 0;
pipeline.pVertexInputState = &vi;
pipeline.pInputAssemblyState = &ia;
pipeline.pRasterizationState = &rs;
pipeline.pColorBlendState = &cb;
pipeline.pTessellationState = NULL;
pipeline.pMultisampleState = &ms;
pipeline.pDynamicState = &dynamicState;
pipeline.pViewportState = &vp;
pipeline.pDepthStencilState = &ds;
pipeline.pStages = info.shaderStages;
pipeline.stageCount = 2;
pipeline.renderPass = info.render_pass;
pipeline.subpass = 0;

res = vkCreateGraphicsPipelines(info.device, NULL, 1,
                                &pipeline, NULL, &info.pipeline);
```
`info.pipeline_layout`、`info.shaderStages`和`and info.render_pass`成员已在以前的章节初始化。这个结构中的余下成员在本章节设置。

---

本篇**Vulkan Samples Tutorial**原文是**LUNAEXCHANGE**中的[Vulkan Tutorial](https://vulkan.lunarg.com/doc/sdk/1.0.42.1/windows/tutorial/html/index.html)的译文。
并非逐字逐句翻译，如有错误之处请告知。O(∩_∩)O谢谢~~



































