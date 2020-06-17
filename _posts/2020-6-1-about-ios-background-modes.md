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

## 注册后台任务标识

首先以刷新任务为例，在 AppDelegate.swift 的 application(_:didFinishLaunchingWithOptions:) 代理方法内加入如下代码。
```swift
import BackgroundTasks
...
    
BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.demo.refresh", using: nil) { task in
    let refreshAppTask = task as! BGAppRefreshTask
    
    refreshAppTask.expirationHandler = {
        
    }
    
    ...
    
    task.setTaskCompleted(success: true)
}

...
```
因为是以刷新任务为例，所以在注册回调中需将任务强转为`BGAppRefreshTask`类型，值得注意的是，系统分配的任务时间有限，所以当系统分配的任务时间即将到期，或者系统提前终结任务时调将会调用`refreshAppTask`的`expirationHandler`回调，所以可以将一些处理未完成任务的操作放入这里。

另外，在任务结束后需调用`setTaskCompleted(success: Bool)`方法，以节省系统资源。

数据处理任务同理，只是任务类型相应的变为`BGProcessingTask`类型，并注意不要将任务标识传错。

<br/>

## 提交后台任务请求

同样先以刷新任务为例，在 AppDelegate.swift 的 applicationDidEnterBackground(_:) 代理方法内加入如下代码。
```swift
...
    
let request = BGAppRefreshTaskRequest(identifier: "com.demo.refresh")
request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
        
do {    
    try BGTaskScheduler.shared.submit(request)
    
} catch {
    print(error)
}

...
```
其中`earliestBeginDate`表示最早开始执行背景任务的时间，可以理解为至少应用进入后台多久之后，才可以执行预定的后台任务，上述代码中为至少应用进入后台十五分钟之后，才可以执行后台任务。

值得注意的有两点，一是在执行`submit`操作时，会阻塞主线程。二是如果需要后台持续刷新，可将此段代码放入注册后台任务标识的回调中。

数据处理任务的请求也差不多，只是请求类型需要为`BGProcessingTaskRequest`，但是数据处理任务请求有两个额外的属性可设置。一是`requiresNetworkConnectivity`，表示是否需要联网，默认为否。二是`requiresExternalPower`，也就是是否只能在充电时进行，，默认为否。

<br/>

## 调试

为了调试，首先需要将应用放入一次后台以提交后台任务请求，再次打开应用并在控制台输入`e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"TASK_IDENTIFIER"]`即可模拟执行后台任务，输入`e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateExpirationForTaskWithIdentifier:@"TASK_IDENTIFIER"]`即可模拟后台任务时间到期。
