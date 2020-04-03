---
layout: post
title: 使用 Swift 开发 iOS 限制输入字数与特殊字符的巧妙方法
image: '/images/Develop.jpg'
---

在 iOS 开发中，时常需要限制用户的输入行为，比如限制用户只能输入数字，或是只能输入中文、英文等等。虽说可以通过设置键盘类型来简单规避掉一些不必要的麻烦，但本文将着重从代码层面以`UITextField`为例，介绍如何限制输入字数与特殊字符。

<br/>

## 添加监听事件

限制`UITextField`输入内容的第一步就是得知`UITextField`中内容的变化。我们可以通过以下代码为`textField`添加监听事件：
```swift
...
    
private let textField = UITextField()

override func viewDidLoad() {
    super.viewDidLoad()
    self.textField.addTarget(self, action: #selector(textFieldDidEdit), for: .editingChanged)
    ...
}

@objc private func textFieldDidEdit() {
    ...
}

...
```
当`textField`中的内容发生变化时，将会调用`textFieldDidEdit()`方法。动态监听`textField`内容变化不仅可以限制用户手动输入的内容，还可以限制复制粘贴的内容。

<br/>

## 补全监听事件

在补全此监听事件前，还需定义一个全局变量`qualifiedString`，代码如下：
```swift
private var qualifiedString = ""
```
该变量用来暂存`textField`中的符合条件的内容，当`textField`中所输入的内容符合条件时，我们不对`textField`进行输入限制，并同时将`textField`中的内容赋值给`qualifiedString`，而当`textField`中所输入的内容不符合条件时，对`textField`的输入进行限制，即用户输入无效，并将`qualifiedString`作为`textField`中的内容保持不变。`textFieldDidEdit()`方法完整代码如下：
```swift
@objc private func textFieldDidEdit() {
    // 1
    guard self.textField.markedTextRange == nil else { return }

    // 2
    guard let text = self.textField.text, text.isEmpty == false else {
        self.qualifiedString = ""
        return
    }
                        
    // 3
    if self.isIncludeSpecialCharacters(in: text) || self.isBeyondNumberOfMaxInput(in: text) {                
    self.textField.text = self.qualifiedString

    } else {
        self.qualifiedString = text
    }
}
```
接下来我将一步步解释这些代码的作用：
1. 判断`textField`是否存在高亮区域，即当用户使用拼音等输入法进行输入时，还没有确定输入选词，拼写时的产生英文不算入输入内容。

2. 判断`textField`中是否有内容，只有当`textField`中存在内容时代码才会继续运行。

3. 代码运行到这里将会报错，因为我们还未定义`isIncludeSpecialCharacters(in: String)`与`isBeyondNumberOfMaxInput(in: String)`方法，后文将会介绍这些的方法的定义以及实现。这段代码会判断`textField`中是否包含特殊字符，或者`textField`中的内容长度是否超过所定义的最大输入数量，如果`textField`中的内容满足上述任意条件，则`qualifiedString`替代`textField`中的内容；否则视`textField`中的内容符合条件，并对`qualifiedString`赋值。

### 处理特殊字符

本文假设中文、英文以外的所有字符皆为特殊字符。于是使用正则表达式对`textField`中的对每一个字符进行判别，并计算特殊字符数量`numberOfCharacters`，只要当`numberOfCharacters`不为零即视为存在特殊字符。代码如下：
```swift
private func isIncludeSpecialCharacters(in string: String) -> Bool {
    let pattern = "[^A-Za-z\\u4E00-\\u9FA5\\d]"
    let expression = try! NSRegularExpression(pattern: pattern, options: .allowCommentsAndWhitespace)
    let numberOfCharacters = expression.numberOfMatches(in: string, options: .reportProgress, range: NSMakeRange(0, (string as NSString).length))
        
    return numberOfCharacters == 0 ? false : true 
}
```

### 处理输入数量限制

本文假设最大输入数量限制为 16 个英文字符或 8 个中文字符，一个中文字符相当于两个英文字符。而在不同编码的字符系统中，中文、英文长度都不同，所以定义变量`count`用来表示中文、英文的字符个数，并以此与最大输入数量进行比较。代码如下：
```swift
private func isBeyondNumberOfMaxInput(in string: String) -> Bool {
    var count = 0
        
    for char in string {
        let lengthOfCharacter = "\(char)".lengthOfBytes(using: .utf8)
            
        // 英文
        if lengthOfCharacter == 1 {
            count = count + 1
        }
                
        // 中文
        else if lengthOfCharacter == 3 {
            count = count + 2
        }
    }

    return count <= 16 ? false : true 
}
```

<br/>

## 注意事项

虽说以上代码已满足限制输入字数与特殊字符的基本要求，但当用户复制的内容中包含特殊字符，或是复制粘贴后字数超出最大输入限制的话，会导致用户所复制的内容完全无法粘贴到`textField`中。解决这个问题的办法是，将是否包含特殊字符与是否超出最大输入限制分开判断，并分别处理。例如，当`textField`中的内容包含特殊字符时，可以只删除掉特殊字符后，再将`textField`中的内容赋值给`qualifiedString`，当`textField`中的内容超出最大输入限制时，按最大输入限制截取`textField`中的内容并赋值给`qualifiedString`，具体代码因为我懒就不再展示。
