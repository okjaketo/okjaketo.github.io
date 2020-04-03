---
layout: post
title: 介绍 GCD 中的线程，队列，延迟执行，线程阻断，信号量，任务组与任务对象
image: '/images/Develop.jpg'
---

GCD 全称 Grand Central Dispatch，是 iOS 开发中最常使用的一种管理多线程的方式，也是苹果公司最为推崇的一种，GCD 最大的优点在于它的简单方便，虽然可能不如其他多线程管理方式那样灵活，但也能适用于绝大部分多线程中的情况。在 GCD 层面没有线程的概念，只有队列。任务都是以闭包的形式提交到对列上，然后 GCD 会自动创建线程去执行这些任务。

声明：笔者自身对 GCD 也是初学，而本文介绍的 GCD 知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

## 线程

一个程序将执行某个操作的时候，就会将这个操作放在程序中的线程里去执行，一个程序可以拥有多个线程。因为有时程序需要同时执行多个操作，所以也就可以拥有多个线程。

线程可以是同步的，也可以是异步的。同步任务不会开启新的线程，所有任务会在当前线程中，按顺序一个个执行，执行完一个再执行下一个。异步任务彼此独立，互不影响，各自执行各自的任务，无需等待其他任务完成，在绝大多数情况下，执行异步任务时会创建新的线程。

<br/>

## 队列
### 与线程的关系
线程是代码执行的路径，队列则是用于保存以及管理任务的，线程负责去队列中取任务进行执行。 

### 分类
队列主要分为串行队列和并行队列两类：在串行队列中，任务按照其在队列中的顺序被调度，前一个任务执行完毕，才会开始执行下一个任务，串行队列一次只能执行一个任务。而在并行队列中，一次能同时执行多个任务，只要有空闲的线程，队列就会调度当前任务，交给线程去执行。并行是 CPU 的多核芯同时执行多个任务。

需要注意的是，并发，并发是指单核 CPU 在同一时间间隔内交替执行两个或多个任务。并行需要并发，但并发不能保证并行。并发是关于结构，而并行是关于执行。

在串行队列和并行队列的基础上，队列还可以继续细分为三类：
* 主队列：
```swift
DispatchQueue.main
```
主队列在主线程上运行，是一个专门用来在主线程上调度任务的串行队列。所有的 UI 更新都必须放在主队列中。

* 全局队列：
```swift
DispatchQueue.global()
```
全局队列是一个整个系统共享的并发队列。在使用多线程开发时，如果对队列没有特殊需求，在执行异步任务时，可以直接使用全局队列。

* 自定义队列：
```swift
let customQueue = DispatchQueue(label: String, qos: DispatchQoS = .unspecified, attributes: DispatchQueue.Attributes = [], autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit, target: DispatchQueue? = nil)
```
自定义队列中的请求实际上最终位于其中一个全局队列中。各参数作用：

  * `label`：队列唯一标识符，一般用 Bundle Identifier 类似的命名方式，将域名翻转，例如：com.xxx.xxx.queue。

  * `qos`：队列任务优先级，的全称是 Quality Of Service，通常使用 QoS 为以下四种，从上到下优先级依次降低：
    1. `.userInteractive`： 和用户交互相关，优先级最高。
    2. `.userInitiated`： 需要立刻的结果，比如 Push 一个 ViewController 之前的数据计算。
    3. `.utility`： 可以执行很长时间，再通知用户结果。比如下载一个文件，给用户下载进度。
    4. `.background`： 用户不可见，比如在后台存储大量数据。

  * `attributes`：队列属性。队列是默认是串行。你可以这样传入参数 [.option1, .option2]，当传入参数为 `.concurrent` 时，队列为并行的。当传入参数为 `.initiallyInactive` 时队列任务不会自动执行，需要开发者手动执行 `.activate()` 方法触发。

  * `autoreleaseFrequency`：自动释放频率。用来设置负责管理任务内对象生命周期的 autorelease pool 的自动释放频率。包含三个类型：
    1. `.inherit`：继承目标队列的该属性。
    2. `.workItem`：跟随每个任务的执行周期进行自动创建和释放。
    3. `.neve`：不会自动创建 autorelease pool，需要手动管理。

  * `target`：指定目标队列。即队列中的任务运行时实际所在的队列。目标队列最终约束了队列的优先级等属性。在程序中手动创建的队列最后都指向了系统自带的主队列或全局并发队列。如果此队列是一个串行队列，那么将此队列的闭包提交到另一个串行队列，并且此队列的目标队列是不同的串行队列，则不会与提交到目标队列的闭包或具有相同目标队列的任何其他队列同时调用该闭包。

### 附注
全局队列，和在提交闭包时也可以指定任务优先级：
```swift
let globalQueueWithQos = DispatchQueue.global(qos: .userInteractive)

globalQueueWithQos.async(qos: .background) {
    //在 QoS 为 background 下运行
}
```

<br/>

## 延迟执行

GCD可以通过`.asyncAfter()`来提交一个延迟执行的任务，比如：

```swift
let deadline = DispatchTime.now() + 2.0
print("Start task")

DispatchQueue.global().asyncAfter(deadline: deadline) { 
    print("End task")
}

// 或者

DispatchQueue.global().asyncAfter(wallDeadline: deadline) { 
    print("End task")
}
```

他们的区别在于，`DispatchTime` 的精度是纳秒，`DispatchWallTime` 的精度是微秒。

<br/>

## 线程阻断
假设有一个并发的队列用来读写一个数据对象。如果这个队列里的操作是读，那么可以同时进行。如果有写的操作，则必须保证在执行写入操作时，不会有读取操作在执行，必须等待写入完成后才能读取，否则就可能会出现读到的数据不对。这个时候我们会用到 `.barrier`。

当在全局队列中时，带有`.barrier`标志的任务，将会最后执行。在自定义并行队列中时，执行到带有`.barrier`标志的任务时，线程将被阻塞（可视为串行队列），此时只执行带有`.barrier`标志的任务，当带有`.barrier`标志的任务执行完成时，才会继续并行执行后续任务。

```swift
let queue = DispatchQueue.global()
// 或者
let queue = DispatchQueue.init(label: "com.xxx.xxx", qos: .default, attributes: .concurrent)

queue.async {
    sleep(2)
    print("Start task 1")
}
        
queue.async(flags: .barrier) {
    sleep(2)
    print("Start task 2")
}
        
queue.async {
    sleep(2)
    print("Start task 3")
}
```

<br/>

## 信号量
信号量的使用非常的简单：
1. 首先创建一个初始数量的信号对象
2. 使用`.wait()`方法让信号量减 1，再安排任务。如果此时信号量仍大于或等于 0，则任务可执行，如果信号量小于 0，则任务需要等待其他地方释放信号。
3. 任务完成后，使用`.signal()`方法增加一个信号量。

信号量是传统计数信号量的封装，用来控制资源被多任务访问的情况。简单来说，如果我只有两个 USB 端口，如果来了三个 USB 请求的话，那么第3个就要等待，等待有一个空出来的时候，第三个请求才会继续执行。

```swift
let queue = DispatchQueue.global()
let semaphore = DispatchSemaphore(value: 2)
        
queue.async {
    semaphore.wait()
    print("Start task 1")
    semaphore.signal()
}
            
queue.async {
    semaphore.wait()
    print("Start task 2")
    semaphore.signal()
}
            
queue.async(flags: .barrier) {
    semaphore.wait()
    print("Start task 3")
    semaphore.signal()
}
```
在串行队列上使用信号量要注意死锁的问题。

<br/>

## 任务组
任务组相当于一系列任务的松散集合，它可以来自相同或不同队列，扮演着管理者的角色。它可以通知（`.notify()`）外部队列，组内的任务是否都已完成。或者阻塞（`.wait()`）当前的线程，直到组内的任务都完成。任务组更适合集合异步任务（如果都是同步任务，直接使用串行队列即可）。

### 创建任务组
创建的方式相当简单，无需任何参数：
```swift
let group = DispatchGroup()
```

### 将任务加入到任务组中
有两种方式加入任务组：

1. 添加任务时指定任务组
```swift
let queue = DispatchQueue.global()
            
queue.async(group: group) {
    print("Start task 1")
    sleep(2)
    print("End task 1")
}
            
queue.async(group: group) {
    print("Start task 2")
    sleep(4)
    print("End task 2")
}
```

2. 使用 `.enter()`、`.leave()` 配对方法，标识任务加入任务组。
``` swift
let queue = DispatchQueue.global()
            
group.enter()
queue.async() {
    print("Start task 1")
    sleep(2)
    print("End task 1")
    group.leave()
}
            
group.enter()
queue.async() {
    print("Start task 2")
    sleep(4)
    print("End task 2")
    group.leave()
}
```
两种加入方式在对任务处理的特性上是没有区别的，只是便利之处不同。如果任务所在的队列是自己创建或引用的系统队列，那么直接使用第一种方式直接加入即可。如果任务是由系统或第三方的 API 创建的，由于无法获取到对应的队列，只能使用第二种方式将任务加入组内。

### 任务组通知
等待任务组中的任务全部完成后，可以统一对外发送通知，有两种方式：
1. `.notify()`方法，它可以在所有任务完成后通知指定队列并执行一个指定任务，这个通知的操作是异步的：
```swift
group.notify(queue: queue) {
    print("All tasks done")
}
```

2. `.wait()`方法，它会在所有任务完成后再执行当前线程中后续的代码，因此这个操作是起到阻塞的作用：
```swift
group.wait()
print("After all tasks done.")
```

`.wait()`方法中还可以指定具体的时间，如果在这个时间之内，`group.wait()`之前的代码已经执行完，那就阻塞线程，等到时间结束再执行后面的代码；如果在这个时间之内，`group.wait()`之前的代码还没有执行完，那也开始执行后面的代码，同时前面的代码，继续执行。
```swift
let timeout = DispatchTime.now() + 2.0
group.wait(timeout: timeout)
```

如果设置 `timeout` 为 `.distantFuture` 的话，那么就和 Barrier 函数一样，会阻塞线程，并一直等待之前的代码执行完，才执行后面的代码。

<br/>

## 任务对象
在队列和任务组中，任务实际上是被封装为一个任务对象。任务封装最直接的好处就是可以取消任务。

### 创建任务
```swift
let workItem = DispatchWorkItem(qos: .default, flags: [DispatchWorkItemFlags()]) {
    print("Start task")
}
```
DispatchWorkItemFlags，它有两组静态属性:
* 执行情况
  * `.assignCurrentContext`: 为闭包分配创建闭包时的执行上下文属性。
  * `.barrier`: 标记任务为栅栏任务，提交至并行队列时生效，如果直接运行该任务对象则无此效果。
  * `.detached`: 取消闭包与当前执行上下文属性的关联。

* QoS 覆盖信息
  * `.enforceQoS`: 将自己的优先级覆盖掉队列的。
  * `.inheritQoS`: 继承自队列的优先级。
  * `.noQoS`: 没有优先级。

### 执行任务
执行任务时，调用任务项对象的 perform() 方法，这个调用是同步执行的：
```swift
workItem.perform()
```
或则在队列中执行：
```swift
queue.async(execute: workItem)
```

### 取消任务
在任务未实际执行之前可以取消任务，调用 cancel() 方法，这个调用是异步执行的：
```swift
workItem.cancel()
```

