---
layout: post
title: 以配置信息（UserDefaults）的形式存储应用数据
image: '/images/Develop.jpg'
---

存储配置信息，可以使用 `UserDefaults` ，这是一种最简单的储存数据的方法，适合用来储存轻量级的数据，UserDefaults 支持储存的数据类型只有 `Data`，`String`，`Number`，`Date`，`Array` 和 `Dictionary` 这几种。

声明：笔者自身对 `UserDefaults` 也是初学，而本文介绍的 `UserDefaults` 知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

## 储存
储存值为“Hello” 的字符串：
```swift
UserDefaults.standard.set(”Hello“, forKey: “helloString”)
```
而如果准备将储存的数据在主程序（Container）和其他扩展（Extensions）之间传递，则需要在 `UserDefaults` 中添加 `Group` 字符串：
```swift
UserDefaults.init(suiteName: “group.com.xxx.xxx”)?.set(“hello”, forKey: “helloString”)
```

<br/>

## 加载
加载 Key 值为“helloString” 的数据：
```swift
let helloStr = UserDefaulte.standard.object(forKey: “helloString”)

print(helloStr)
// Output: Hello
```
同样在其他扩展（Extensions）中，也需要添加 `Group` 字符串，才能加载相应数据。
```swift
let helloString = UserDefaults.init(suiteName: “group.com.xxx.xxx”)?.value(forKey: “helloString”)

print(helloStr)
// Output: Hello
```
