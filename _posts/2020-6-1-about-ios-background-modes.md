---
layout: post
title: 关于 iOS 后台任务
image: '/images/Develop.jpg'
---

iOS 的后台任务不仅仅可以让应用在进入后台之后，申请额外的系统资源去执行未完成的任务，还可以通过 iOS 的深度学习预测用户打开应用的时间，并在那之前提前执行预定的后台任务，使用应用更快响应。

<br/>

## 添加背景模式

![5](/images/about-ios-background-modes/5.png)

由于本文将分别介绍后台刷新与后台数据处理，所以在背景模式中分别勾选了 Background fetch 与 Background processing，在实际的开发中应该按需选择。

<br/>

## 添加后台任务标识

每个后台任务都需要一个标识，系统才能以此判断需要执行哪个任务。

在 Info.plist 文件中，添加 Permitted background task scheduler identifiers 选项，并在此选项下添加所需执行后台任务的标识。

![6](/images/about-ios-background-modes/6.png)

如图所示，我添加了两个任务标识，分别表示刷新任务与数据处理任务。

<br/>
