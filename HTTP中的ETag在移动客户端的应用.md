"=========== Meta ============
"StrID : 14
"Title : HTTP中的ETag在移动客户端的应用
"Slug  : http-etag
"Cats  : 移动端
"Tags  : HTTP, Net
"Date  : 20170207T15:55:21
"=============================
"EditType   : post
"EditFormat : Markdown
"========== Content ==========
 
绝大多数移动客户端在设计网络模块时，都会选用HTTP作为客户端和服务端通信的网络协议。随着业务的不断发展以及用户量的持续增长，整个客户端的稳定性和性能会逐渐成为关注的焦点，其中网络的性能优化更是重中之重，本文介绍的 *ETag* 缓存技术，可以在缓存数据的同时做到数据的实时更新，适用于对数据实效性要求较高的业务。

<!--more-->

## 基本原理和概念

相同的两次请求返回的结果相同时，第一次返回的结果缓存在客户端，第二次服务端不再返回结果，仅返回一个特殊的状态码，告诉客户端第二次请求的结果与上次相同，可以直接使用上次返回的数据。

实现中，会用到HTTP头中的两个字段：

- **ETag** 返回应答数据的标记，服务端生成发送给客户端
- **If-None-Match** 同样的请求，上一次返回的 ETag 值

## 交互过程

- 服务端在将数据发送给客户端之前，首先计算应答数据的摘要（通常是MD5），把计算结果作为 *ETag* 的值，和数据一同发送给客户端。
- 客户端收到应答数据后，检测 HTTP Header 中是否有 *ETag* 字段，如有则缓存应答数据和 *ETag* 的值。
- 客户端再次发起同一请求时，读取上次缓存的 *ETag* 值，将其作为 *If-None-Match* 的值，并与请求数据一同发送给服务端。
- 服务端收到请求，执行请求，在把应答数据返回给客户端之前计算摘要，并与客户端上报的摘要比较，如果两次摘要相同，说明本次的应答数据与上一次请求的应答数据相同，且客户端已缓存该数据，则简单返回304错误。
- 客户端收到304错误，直接读取本地缓存的数据返回给调用网络模块的业务方。

交互过程总结如下图：

![ETag](http://7xnua6.com1.z0.glb.clouddn.com/2017/02/etag.png)

## 代码实现

本文的示例代码使用 `NSURLSession` 实现，由于 `NSURLSession` 完善的缓存策略，为了演示 *ETag* 的用法，需要先关闭缓存。

```swift
let config = NSURLSessionConfiguration.defaultSessionConfiguration()
config.requestCachePolicy = NSURLRequestCachePolicy.ReloadIgnoringCacheData
let session = NSURLSession(configuration: config)
```

`NSURLSessionConfiguration` 定义了 `NSURLSession` 在上传和下载时的行为及策略，我们指定了请求的缓存策略为 `ReloadIgnoringCacheData`, 意思就是在请求时不使用本地缓存。

创建请求的 `NSURLRequest` 对象：

```swift
let url = NSURL(string: "http://www.joywek.com/50x.html")!
let request = NSMutableURLRequest(URL: url)
```

上面会下载指定静态页面的内容，这个页面是放在 Nginx 的服务器上，Nginx 默认会对应答数据计算 *ETag*。当然，在实际的应用中请求的都是动态数据，服务器要动态计算 *ETag* 的值。

设置 HTTP Header 中的 *If-None-Match* 字段：

```swift
if let tag = self.findTagByURL(url) {
	request.addValue(tag, forHTTPHeaderField: "If-None-Match")
}
```

在发起请求时，先检查相同的请求是否存在 *ETag*，如果存在就，就意味着上次请求的应答数据已缓存。

发起请求并处理应答数据：

```swift
self.dataTask = session.dataTaskWithRequest(request,
    completionHandler: { (var data, response, error) -> Void in
        data = self.handleResponse(response!, data!, request)
})
self.dataTask?.resume()
```

如果 HTTP 返回的状态码是 200，说明是服务器正常返回数据，此时记录 *ETag* 的值并缓存应答数据：

```swift
if (resp.statusCode == 200) {
    self.etags[response.URL!] = resp.allHeaderFields["ETag"] as? String
    let cachedResponse = NSCachedURLResponse(response: resp, data: data)
	NSURLCache.sharedURLCache().storeCachedResponse(cachedResponse, forRequest: request)
    return data
}
```

如果返回 304，说明应答数据没有变化，与上次请求的一样，则直接返回缓存中的数据：

```swift
else if (resp.statusCode == 304) {
    let cachedResponse = NSURLCache.sharedURLCache().cachedResponseForRequest(request)
    return cachedResponse?.data
}
```

## 关于演示 Demo

地址：https://github.com/vimkoo/ETagExample

