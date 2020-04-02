---
layout: post
title: Swift 中的空字符串
image: '/images/Develop.jpg'
---

文：K Harrison，译： okjaketo，校对： okjaketo，原文：[https://useyourloaf.com/blog/empty-strings-in-swift/](https://useyourloaf.com/blog/empty-strings-in-swift/)，本文基于创作共同协议（BY-NC），由 okjaketo 在东方之胱发布。

在 Swift 中，如何判断空字符串，取决于如何定义“空”。你可以认为一段长度为零的字符串是空字符串，也可以认为可选值为`nil`的字符串是空字符串，那一段只包含空格的字符串是不是空字符串那呢？现在就让我们来模拟 Swift 中字符串为空的各种情况。

<br/>

## 使用 isEmpty

在 Swift 中，字符串是字符的合集，而字符串所遵循的`Collection`协议，已经为我们准备好了字符串是否为空的答案：
```swift
var isEmpty: Bool { get }
```
我们在`Collection.swift`的源码中，可以看到该协议具体做了什么：
```swift
public var isEmpty: Bool {
    return startIndex == endIndex
}
```
由此可见，如果一段字符合集中起始索引与结束索引相等，也就代表这段字符合集为空，在字符串中则表示为：
```swift
"Hello".isEmpty  // false
"".isEmpty       // true
```
注意：别用`count`是否等于零来判断字符串是否为空，因为这将迭代整个字符串：
```swift
// 别用这种方法判断字符串是否为空
myString.count == 0
```

<br/>

## 关于空格

有时候我不仅需要判断是否为空字符串，还需要判断是否是只包含空格的字符串。例如，我想在判断是否为空字符串时，让以下字符串都返回`true`（译者注：以下字符串在使用`isEmpty`进行判断时皆返回`false`）：
```swift
" "        // 空格
"\t\r\n"   // tab, 回车, 换行
"\u{00a0}" // Unicode 中的不可分空格
"\u{2002}" // Unicode 中的英文空格
"\u{2003}" // Unicode 中的 em 空格
```
许多人通过去掉空格再判断字符串是否为空。而在 Swift 5 中，我们可以利用[字符属性（Character Properties）](https://useyourloaf.com/blog/character-properties-in-swift-5/)对空格进行直接判断。我们可以像这样写：
```swift
func isBlank(_ string: String) -> Bool {
    for character in string {
        if !character.isWhitespace {
            return false
        }
    }
    return true
}
```
但仍有更简单的办法对字符串中每个字符进行判断，就是使用`allSatisfy`（译者注：Swift 4.2 中新推出来`allSatisfy`方法，该方法运行一个状态闭包（Condition Closure），如果传递给这个闭包后，所有元素都返回`true`，那么该方法就返回`true`），为`String`类型添加扩展：
```swift
extension String {
    var isBlank: Bool {
        return allSatisfy({ $0.isWhitespace })
    }   
}
```
这样看起来就很美妙了：
```swift
"Hello".isBlank        // false
"   Hello   ".isBlank  // false
"".isBlank             // true
" ".isBlank            // true
"\t\r\n".isBlank       // true
"\u{00a0}".isBlank     // true
"\u{2002}".isBlank     // true
"\u{2003}".isBlank     // true
```

<br/>

## 关于可选值
我们可以继续扩展上述判断方式，使其可以对可选值进行判断。以下是解包为`String`类型可选择扩展：
```swift
extension Optional where Wrapped == String {
    var isBlank: Bool {
        return self?.isBlank ?? true
    }
}
```
如果可选字符串为`nil`，则返回`true`；否则，我们使用以前添加到字符串中的`isBlank`属性进行测试。我们现在可以写：
```swift
var title: String? = nil
title.isBlank            // true
title = ""               
title.isBlank            // true
title = "  \t  "               
title.isBlank            // true
title = "Hello"
title.isBlank            // false
```
从今往后，当你需要判断字符串是否为空时，就再也用不到`isEmpty`了。
