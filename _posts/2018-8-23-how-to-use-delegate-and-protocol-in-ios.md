---
layout: post
title: 在 iOS 开发中使用代理与协议
image: '/images/Develop.jpg'
---

创建和使用代理主要分为五个步骤：定义协议，创建实现协议的类的引用，告诉该类谁将实现其协议，将任务传递给实现协议的类，并在该类实现协议中的所有方法。

声明：笔者自身对代理与协议也是初学，而本文介绍的代理与协议知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

### 1. 定义协议
```swift
import ...
protoco ProtocolName: class {
    func FunctionName(controller: CurrentViewController, ...)
    ...
}
class CurrentViewController: ...{...}
```
定义一个协议时，只需要列出需要实现的方法的名称，而不用方法的完整代码。如果一个类是这个类的代理，那么这个类需要实现协议中的所有方法。

### 2. 创建实现该协议的类的引用
```swift
var delegate: ProtocolName?
```
创建一个名为 delegate 的变量，该变量表示实现该类中的协议的类。

<br/>

### 3.告诉该类，哪个类会实现它的协议
```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if segue.identifier == “SomeIdentifier” {
        let controller = segue.destination as! CurrentViewController
        ...
        controller.delegate = self
    }
}
```
通过重写 `prepare(for:, sender:)` 方法，将定义该协议的类中的 delegate 变量值赋为实现该协议的这个类。所以该类现在能知道谁是它的代理。

### 4. 将任务传递给实现该协议的类
```swift
func someFunction (...) {
    ...
    delegate?.CurrentViewController(self, ...)
}
```
在定义协议的类中，当执行包含 “delegate?…” 的语句的方法时，实现该协议的类将接收来定义该协议的类的通知。

### 5. 实现协议中的所有方法
```swift
func FunctionName(controller: CurrentViewController, ...) {
    ...
}
```
接收到通知后，它需要开始实现它所遵循的协议中的所有方法。

代理与协议的基本逻辑与事件就是这样。
