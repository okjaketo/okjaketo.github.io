---
layout: post
title: 当我们谈论离屏渲染时，我们在谈论什么
image: '/images/Develop.jpg'
---

本文将会介绍有关离屏渲染的基础知识，但在正式开始之前，先粗略介绍下 iPhone 的屏幕显示原理，这样有助于更好地理解离屏渲染。图像显示到屏幕上，大概会经历以下几个步骤：

1. CPU 告诉 GPU 需要显示的内容
2. GPU 按照 CPU 的要求，将 Layer 渲染进 Frame Buffer（帧缓冲区）
3. 渲染完成后， Video Controller（视频控制器）把渲染结果（作为一帧）显示到屏幕上

由于 iPhone 的刷新率是 60 Hz，所以上述过程正常情况下每秒会发生 60 次。这也就是大概的 iPhone 屏幕显示原理。

<br/>

## 离屏渲染

前文中提到「GPU 会将 Layer 渲染进 Frame Buffer」，这是正常的渲染流程，被称为 On-Screen Rendering（当前屏幕渲染）。而离屏渲染与正常渲染流程的唯一不同就在于，当 GPU 在渲染时，由于一些原因，不能将 Layer 渲染进 Frame Buffer，而是需要创建一个 Offscreen Buffer（离屏缓冲区）来参与渲染，然后再将渲染结果放入 Frame Buffer，这个过程就被称为 Off-Screen Rendering（离屏渲染）。

因为离屏渲染需要创建新缓冲区、多次切换上下文环境（从 On-Screen 切换到 Off-Screen）以及每秒将会进行 60 次这样的繁重操作，所以大量的离屏渲染会引起性能问题、降低显示帧率、造成卡顿。

<br/>

## 为什么会发生离屏渲染

关于为什么会发生离屏渲染，其实我没有找到官方的解释，但是因为 GPU 在渲染时遵循画家算法，也就是说对于每一层 Layer，都会被依次渲染进 Frame Buffer，后一层不断覆盖前一层，被覆盖的地方的数据会永久丢失，并且切不可逆。

根据这一点可以推测出：GPU 在渲染每一层 Layer 时，绝对无法得知它的上一层和下一层 Layer 的信息，也无法修改它们，所以在遇到无法依次渲染的情况时，就不得不新建一个 Offscreen Buffer 来储存渲染的中间结果，从而发生离屏渲染。

<br/>

## 造成离屏渲染的原因与解决办法

### 1. `shadow`（阴影）
因为`shadow`必须知道其上层 Layer 的形状才能渲染，而上层 Layer 又必须等其下层 Layer 渲染完成才能渲染，所以就不得不新建一个 Offscreen Buffer 参与渲染。

解决办法就是设置 Layer 的`.shadowPath`，这样可以使 GPU 在渲染阴影 Layer 时，不用知道其上层 Layer 形状就可以完成阴影路径的渲染。

### 2. `cornerRadius` + `clipsToBounds`（圆角与裁切圆角以外内容）
在 Layer 被渲染时，其上层 Layer 还没有被渲染，所以无法完成裁切圆角以外内容，所以也需要新建一个 Offscreen Buffer 参与渲染。

解决办法是使用`UIBezierPath`与`CAShapeLayer`设置圆角。

### 3. `group opacity`（群组透明）、`mask`（遮罩）、`UIBlurEffect`（高斯模糊）与`allowsEdgeAntialiasing`（抗锯齿）
在所有 Layer 的渲染完成后，再应用透明度，与分别为每层 Layer 应用透明度再渲染得到的结果是不同的，与透明度相关以及抗锯齿的渲染都无可避免的需要一个 Offscreen Buffer 参与渲染。

### 4. `shouldRasterize`（栅格化）
`shouldRasterize`相当于主动进行离屏渲染，目的是为了将渲染结果保存在一起，使下一帧可以直接复用，避免重复渲染。

<br/>

## 后续
关于离屏渲染还有很多细节，本文并没有提及，感兴趣的读者可以从 CPU 渲染与 `shouldRasterize`属性深入研究下去。
