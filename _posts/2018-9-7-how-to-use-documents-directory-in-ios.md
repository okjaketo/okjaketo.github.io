---
layout: post
title: 在文件沙盒中读取与写入应用数据
image: '/images/Develop.jpg'
---

iOS 应用程序在安装时，会创建属于自己的沙盒文件，应用程序不能直接访问其他应用程序的沙盒文件。应用程序中所有的非代码文件都保存在沙盒中，比如图片、声音、属性列表，sqlite 数据库和文本文件等。沙盒的的根目录有三个文件夹分别是：

* Tmp，此目录保存应用程序运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录

* Library， 此目录下有两个子目录：Caches 和 Preferences：

    * Caches 是用来保存应用程序运行时生成的需要持久化的数据，这些数据一般存储体积比较大，又不是十分重要，比如网络请求数据等。这些数据需要用户负责删除，iTunes 同步设备时不会备份该目录。

    * Preferences 用来保存应用程序的所有偏好设置，iOS 的设置应用会在该目录中查找应用的设置信息。在 Preferences 下不能直接创建偏好设置文件，而是应该使用 `UserDefaults` 来取得和设置应用程序的偏好.iTunes 同步设备时会备份该目录。

* Documents， 此目录下一般保存应用程序本身产生文件数据，例如游戏进度，绘图软件的绘图等，iTunes 备份和恢复的时候，会包括此目录。在此目录下不能保存从网络上下载的文件，否则应用无法上架。

而本文所描述的存储方式就是在 Documents 目录下的储存方式。

声明：笔者自身对文件沙盒也是初学，而本文介绍的文件沙盒知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

## 储存
储存的步骤主要分为两步。第一步是确定存储数据的文件的位置。第二步是将数据储存在文件中。

### 获取存储数据的文件
```swift
func dataDocumentsDirectory() -> URL {
    return FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
}
```

此方法的目的是构建存储数据的文件的路径。此方法没有标准名称，您可以使用任何您喜欢的名称。

```swift
func dataFilePath() -> URL {
    return dataDirectory().appendingPathComponent("SomeName.plist")
}
```
该方法是在上一个方法中所构建的路径中创建储存文件。这个文件被命名为 “SomeName.plist”，以后需要存储的所有数据都可以存储在这个文件中。

### 将数据储存至文件
接下来的方法会将任何的数据内容，转换为二进制数据，然后写入该存储数据文件。首先需要创建一个 `PropertyListEncoder` 的实例：`encoder`，该实例会将要存储的任何数据编码成某种可以写入该文件的二进制数据格式。

```swift
func saveData(_ anyData: anyType) {
    let encoder = PropertyListEncoder()
}
```

但是，如果编码方法由于某种原因无法对数据进行编码，则会引发错误。例如，数据不是预期的格式，或者已损坏等。幸运的是，Swift 通过抛出错误来处理某些条件下的错误。在这种情况下，您需要一段代码来捕获错误并进行处理。`do`关键字表示有可能抛出错误的代码块的开始。`try`关键字表示代码所执行的任务可能失败，如果发生这种情况，它将抛出错误。 `catch`语句表示捕获所抛出的错误的代码块，`do`代码块中的任何代码抛出的错误都将被捕获，并执行相应的代码。在这里，我们只是简单地将错误打印到 Xcode 控制台。

```swift
func saveData(_ anyData: anyType) {
    let encoder = PropertyListEncoder()
    do {
        let data = try encoder.encode(anyData)
    } catch {
        print("Encoding Error.")
    }
}
```
如果数据被上一个方法成功编码，则使用 `dataFilePath()` 调用存储文件的路径，将数据写入该文件。写入方法也会导致错误，所以你必须在方法调用之前使用另一个`try`语句。

```swift
func saveData(_ anyData: anyType) {
    let encoder = PropertyListEncoder()
    do {
        let data = try encoder.encode(anyData)
        try data.write(to: dataFilePath(), options: Data.WritingOptions.atomic)
    } catch {
        print("Encoding Error.")
    }
}
```

<br/>

## 加载
储存做好了，但还不够用。所以，我们还要实现加载 “SomeName.plist” 文件的功能。在加载数据的方法中。首先将 `dataFilePath()` 的数据放入一个名为`path`的临时常量中。
```swift
func loadData() {
    let path = dataFilePath()
}
```
再尝试将 SomeName.plist 的内容加载到新的 Data 对象中。`try`关键字尝试创建 Data 对象，但如果失败则返回 `nil`，这就是为什么你把它放在一个`if let`语句中。
```swift
func loadData() {
    let path = dataFilePath()
    if let data = try? Data(contentsOf:path) {
    }
}
```
为什么会失败？如果没有 SomeName.plist，那么显然没有要加载的数据。这是应用程序第一次启动时发生的情况，在这种情况下，您将跳过此方法的其余部分。

然后创建解码器实例：`decoder`。当应用程序找到 SomeName.plist 文件时，您将使用`decoder`从文件加载所有数据。
```swift
func loadData() {
    let path = dataFilePath()
    if let data = try? Data(contentsOf:path) {
        let decoder = PropertyListDecoder()
    }
}
```
解码器将保存的数据加载回原来的位置。这里唯一有趣的数据是传递给解码器的第一个参数。解码器需要知道解码操作的结果是什么类型的数据，并通过指示它将是存储时的数据类型来告诉它。
```swift
func loadData() {
    let path = dataFilePath()
    if let data = try? Data(contentsOf:path) {
        let decoder = PropertyListDecoder()
        do {
            originalData = try decoder.decode([originalType].self, from: data)
        } catch {
            print("Decoding Error.")
        }
    }
}
```
最后，将`saveData()`方法和`loadDate()`方法放在需要调用的地方。例如，当程序开始运行时，结束时之类的。
