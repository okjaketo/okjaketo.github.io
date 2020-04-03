---
layout: post
title: 使用 iOS 原生的 URLSession 进行网络请求
image: '/images/Develop.jpg'
---

应用无论是从服务器获取信息，还是更新社交媒体信息，抑或是下载文件，这都归功于应用核心的 HTTP 网络请求。为了帮助开发者满足网络请求的众多需求，苹果提供了 `URLSession`，这是一个完整的网络 API，用于发送和接收 HTTP/HTTPS 请求。

你可以通过 `URLSessionConfiguration` 来创建 `URLSession`，它有三种形式：

* `.default`：创建全局持久化本地缓存，证书和 cookie 存储对象。
* `.ephemeral`：与默认配置类似，只是所有与任务相关的数据都存储在内存中，可将此视为“私密”任务。
* `.background`：允许在后台执行上传或下载任务。即使应用本身被系统暂停或终止，任务仍会继续。

`URLSessionConfiguration` 还允许你配置其他属性，例如超时值，缓存策略和 HTTP 头部信息等。

`URLSessionTask` 是一个表示网络请求任务的抽象类。网络请求会创建一个或多个任务来获取数据，和上传或下载。这里有四种类型的网络请求任务：

* `URLSessionDataTask`：此任务用于 HTTP GET 请求，以将数据从服务器检索到内存。
* `URLSessionUploadTask`：此任务通常用于 HTTP POST 或 PUT 方法将文件从本地上传到服务器。
* `URLSessionDownloadTask`：此任务用于将文件从服务器下载到临时文件位置。
* `URLSessionStreamTask`：此任务用于建立 TCP/IP 长连接。

您也可以暂停，恢复和取消任务。 `URLSessionDownloadTask` 可以保存暂停时的状态，以至于恢复任务时，不用从头开始。

声明：笔者自身对 `URLSession` 也是初学，而本文介绍的 `URLSession` 知识也都是自己的一些理解，同时尽量不涉及过于原理性的内容，一切以普通工程师实用为目标原则。其中可以想象在很多地方会有理解的错误，还请多包涵。如您发现问题，也往不吝赐教指正，感激不尽。

<br/>

## 组装 URL

在进行任何一项网络请求时，都需要配置 URL，苹果提供 `URLComponents` 以供开发者配置 URL。比起使用纯字符串形式的 URL，用这种方式可以根据 URLComponents 值的内容轻松获取 URL 值，反之亦然。
```swift
func makeURL() -> URL {
    var components = URLComponents()
    components.scheme = "https"
    components.host = "xxx.xxx.xxx"
    components.path = "/path"

    components.queryItems = [URLQueryItem]()
    components.queryItems?.append(URLQueryItem(name: "parameterName1", value: "parameterValue1"))
    components.queryItems?.append(URLQueryItem(name: "parameterName2", value: "parameterValue2"))
            
    return components.url!
}
```

<br/>

## 发起 GET 请求

发起 GET 请求时，必须先创建 `URLSessionTask` 对象和获取 URL 地址。为防止重复发起请求，可在 `dataTask` 的调用 `.resume()` 方法开始每次任务前，调用 `.cancel()` 方法。
```swift
func getRequest() {
    var dataTask: URLSessionTask?
    dataTask?.cancel()

    let requestURL = makeURL()
    let request: URLRequest = URLRequest(URL: requestURL)

    dataTask = URLSession.shared.dataTask(with: request, completionHandler: { (data, response, error) in
        ......
    })
        
    dataTask?.resume()
}
```

另一种发起 GET 请求的方式，同样是调用 `dataTask(with:completionHandler:)` 方法，只不过第一个参数可以直接传入 URL 地址。

<br/>

## 发起 POST 请求

与发起 GET 请求的不同在于，需要指定 `URLRequest` 的请求类型，并且设置 POST 的内容。
```swift
func postRequest() {
    ......

    request.httpMethod = "POST"
    let postData = ["parameterName1": "parameterValue1", "parameterName2": "parameterValue2"]
    let postString = postData.compactMap({ (key, value) -> String in
        return "\(key)=\(value)"
    }).joined(separator: "&")
    request.httpBody = postString.data(using: .utf8))

    dataTask = URLSession.shared.dataTask(with: request, completionHandler: { (data, response, error) in
        ......
    })
        
    ......
}
```

<br/>

## 开始下载
### 发起简单的下载任务
如果不需要获取下载进度，可以就像之前发起 POST 请求时那样调用下载任务的方法：
```swift
func postRequest() {
    ......

    downloadTask = URLSession.shared.downloadTask(with: request) { (url, response, error) in
        print("Finished downloading to \(location).")
        ......
    }
        
    ......
}
```

### 开始下载任务
如果需要监听和及时更新下载进度，那就需要用到 `URLSessionDownloadDelegate`。而引入 `URLSessionDownloadDelegate` 必须要实现的方法是 `urlSession(_:downloadTask:didFinishDownloadingTo:)`，该方法会在下载任务执行完成后被调用。
```swift
extension DownloadViewController: URLSessionDownloadDelegate {
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) { 
        print("Finished downloading to \(location).")
        ......
    }
}
```
但首先，需要初始化一个特别的 `URLSession` 来负责调用代理：
```swift
lazy var downloadsSession: URLSession = {
    let configuration = URLSessionConfiguration.default
    return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}()
```
这里使用默认配置初始化网络任务，并指定代理，这样便可以接收 `URLSession` 的代理事件，用于监听任务进度。将代理队列设置为 `nil` 会导致创建一个串行队列，以执行委派方法和处理程序的所有调用。

然后再使用 downloadsSession 发起一个下载任务：
```swift
var downloadTask: URLSessionDownloadTask = downloadsSession.downloadTask(with: downloadUrl)
downloadTask.resume()
```
当下载任务执行完后，之前提及过的 `urlSession(_:downloadTask:didFinishDownloadingTo:)` 方法会被调用，在这个方法中，所下载的文件将被存储在一个临时区域，如果需要长久使用这个文件的话，需要将它放入应用沙盒中：
```swift
func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) { 
    let documentsPath = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
    let destinationURL = documentsPath.appendingPathComponent(downloadTask.originalRequest?.url.lastPathComponent)

    let fileManager = FileManager.default
    try? fileManager.removeItem(at: destinationURL)
    do {
        try fileManager.copyItem(at: location, to: destinationURL)
    } catch let error {
        print("Could not copy file to disk: \(error.localizedDescription)")
    }

    ......
}   
```
在这段代码中，首先按照所下载文件在服务器中的路径，在本地创建相同的路径，并清空。然后再将文件从临时的储存位置移动到先前创建好的地址。

### 暂停，继续和取消下载任务
取消下载任务的方式十分简单，直接在 `downloadTask` 后调用 `.cancle()` 方法即可。

暂停下载任务和取消下载任务很相似，只是需要在取消下载任务时，保存好已下载的数据：
```swift
downloadTask?.cancel(byProducingResumeData: { data in
    let downloadedData = data
    ......
})   
```
继续下载任务时，如果存在已下载的数据则继续下载，如果没有，则重新开始下载：
```swift
if let resumeData = downloadedData {
    downloadTask = downloadsSession.downloadTask(withResumeData: resumeData)
} else {
    downloadTask = downloadsSession.downloadTask(with: downloadUrl)
}
    
downloadTask.resume()
```
别忘记，无论哪种情况发起何种请求，都需要调用 `resume()` 方法。

### 获取下载任进度
显示下载进度可以有效的提升用户体验：
```swift
func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didWriteData bytesWritten: Int64, totalBytesWritten: Int64, totalBytesExpectedToWrite: Int64) {
    let totalSize = ByteCountFormatter.string(fromByteCount: totalBytesExpectedToWrite, countStyle: .file)
    let downloadProgress = Float(totalBytesWritten) / Float(totalBytesExpectedToWrite)
    ......
}
```
### 在后台执行下载任务
启用后台下载之后，即便是应用在后台或崩溃，除非用户手动终止应用，否则下载仍会继续，这非常适用于大型文件的下载。这是由于当程序没有执行的时候，系统在应用外运行一个单独的守护程序来管理后台传输任务，并在下载任务时将适当的代理消息发送到应用程序。如果应用在这期间被终止，下载任务也将在后台继续执行。任务完成后，守护程序将在后台重新启动应用。重新启动的应用程序将重新创建后台会话，以接收相关的完成代理消息，并执行任何所需的操作，例如将下载的文件保存到磁盘。

想要实现此功能，需要在初始化下载任务时，就指定下载任务的模式：
```swift
lazy var downloadsSession: URLSession = {
    let configuration = URLSessionConfiguration.background(withIdentifier: "backgroundSessionConfiguration")
    return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}()
```
如之前所说，当后台下载任务完成后，应用将会重启，这时候需要一个代理方法来执行这个事件，可在 `AppDelegate.swift` 文件执行 `application(_:handleEventsForBackgroundURLSession:)` 这个方法：
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    var backgroundSessionCompletionHandler: (() -> Void)?
  
    func application(_ application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: @escaping () -> Void) {
        backgroundSessionCompletionHandler = completionHandler
    }
}
```
最后再在 `URLSessionDelegate` 当任务完成的代理方法中执行具体的操作：
```swift
func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
    DispatchQueue.main.async {
        if let appDelegate = UIApplication.shared.delegate as? AppDelegate, let completionHandler = appDelegate.backgroundSessionCompletionHandler {
            appDelegate.backgroundSessionCompletionHandler = nil
            completionHandler()
        }
    }
}
```

<br/>

## 开始上传

发起上传任务和发起 POST 请求很类似，只是将需要上传的文件或数据放在上传任务初始化时。
```swift
func postRequest() {
    ......

    request.httpMethod = "POST"

    let documents =  NSHomeDirectory() + "/Documents/fileName.suffix"
    let data = try! Data(contentsOf: URL(fileURLWithPath: documents))

    uploadTask = URLSession.shared.uploadTask(with: request, from: data) { (data, response, error) in
        ......
    }

    // 或者 

    uploadTask = URLSession.shared.uploadTask(with: request, fromFile: URL(string: documents)) { (data, response, error) in
        ......
    }
        
    ......
}
```
