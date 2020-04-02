---
layout: post
title: 在 iOS 开发中使用键值对监听（KVO）与通知（Notification）
image: '/images/Develop.jpg'
---

声明：笔者自身对 KVO 与 Notification 也是初学，而本文介绍的 KVO 与 Notification 知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

## 注册
### KVO
被监听的对象，必须继承`NSObject`，被监听的变量，必须加上`@objc dynamic`修饰符。
```swift
class Person: NSObject {
    @objc dynamic var age = 0
        
    override init() {
        super.init()
    }
}
```
KVO 注册监听的方式有三种：
```swift
let person = Person()

// 纯注册监听
person.addObserver(self, forKeyPath: "age", options: [.new, .old], context: nil)
            
// 注册监听，并且设置触发动作
person.observe(\Person.age) { (person, change) in
    print(change.newValue)
    print(change.oldValue)
}
            
// 注册监听，并且设置触发动作
person.observe(\.age, options: [.new, .old]) { (person, change) in
    print(change.newValue)
    print(change.oldValue)
}
```

### Notifacation
```swift
let ageKey = Notification.Name("AgeKey")

NotificationCenter.default.addObserver(self, selector: #selector(notificationObserveFunc), name: ageKey, object: nil)

@objc func notificationObserveFunc() {
    print("Notification Observe")
}
```

<br/>

## 触发
### KVO
通过重写`observeValue(forKeyPath:of:change:context:)`，当被监听的值发生改变的时候即可触发事件。
```swift
override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
    print("New: \(change![NSKeyValueChangeKey.newKey])")
    print("Old: \(change![NSKeyValueChangeKey.oldKey])")
}
```

### Notifacation
```swift
NotificationCenter.default.post(name: ageKey, object: nil)
```

<br/>

## 注销
### KVO
```swift
deinit {
    person.removeObserver(self, forKeyPath: "age")
}
```

### Notifacation
```swift
deinit {
    NotificationCenter.default.removeObserver(self)
}
```
