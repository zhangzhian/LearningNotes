# OkHttp：进阶使用详解

在《OkHttp：基本使用详解》中描述了OkHttp的基本使用方法。已掌握了如何使用OkHttp进行基本的网络请求，但这只是最基础的使用，想要在实际的开发项目中应用OkHttp是不够的。

本文《OkHttp：进阶使用详解》主要详细的描述OkHttp的使用时涉及到的相关概念，为后续阅读源码提供基础。

接下来通过《OkHttp：源码详解》一文从源码的角度来阐述OkHttp的设计思想。

## 一、Calls

HTTP客户端的工作是接受您的请求并产生其响应。从理论上讲这很简单，但在实践中却很棘手。

### 请求

每个HTTP请求都包含一个URL，一个方法（如GET或POST）和 header 列表。请求可能还包含body：特定内容类型的数据流。

### 响应

响应使用code（例如200表示成功或404表示找不到），header 和其自己的可选body来回答请求。

### 重写请求

为了确保正确性和效率，OkHttp在传输请求之前会先对其进行重写。

OkHttp可以添加原来的请求不存在headers，包括`Content-Length`，`Transfer-Encoding`，`User-Agent`，`Host`，`Connection`，和`Content-Type`。除非`Accept-Encoding` header已经存在，否则它将添加用于透明响应压缩的标题。如果使用Cookie，OkHttp将在其中添加Cookie标题。

一些请求将具有缓存的响应。当此缓存的响应不是最新的，OkHttp可以执行条件GET来下载更新的响应。这种请求会在header添加`If-Modified-Since`和`If-None-Match`。

### 重写响应

如果使用透明压缩，则OkHttp将删除相应的响应标头`Content-Encoding`和`Content-Length`，因为它们不适用于解压缩的响应正文。

如果有条件的GET成功，则按照规范的指示将来自网络和缓存的响应合并。

### 后续请求

当请求的URL被移动，网络服务器将返回一个响应代码，例如`302`指示文档的新URL。OkHttp将遵循重定向以检索最终响应。

如果响应发出授权质询，OkHttp将要求[`Authenticator`](http://square.github.io/okhttp/4.x/okhttp/okhttp3/-authenticator/)（如果已配置）满足质询。如果身份验证器提供了凭据，则将使用该凭据重试请求。

### 重试请求

有时连接失败：池化连接已断开连接，或者无法访问Web服务器本身。如果其他路由可用，OkHttp将重试该请求。

### Cell

通过重写，重定向，跟进和重试，一个简单请求可能会产生许多请求和响应。OkHttp用于`Call`对这一过程（通过中间请求和响应来满足用户真正的请求）进行建模。若URL重定向或故障转移到备用IP地址，代码将继续起作用。

调用以两种方式之一执行：

- **同步：**您的线程阻塞，直到响应可读为止。
- **异步：**将请求放入任何线程中，并在响应可读时在另一个线程上被调用。

可以从任何线程取消Cell。如果尚未完成，这将使Cell失败。正在写请求body或读取响应body的时消其调用，将抛出`IOException`。

### Dispatch

对于同步调用，需要带上自己的线程，并自己管理并发的请求数量。同时连接过多会浪费资源。太少会损害延迟。

对于异步调用，`Dispatcher`实现最大同时请求数的策略。可以设置每个Web服务器的最大值（默认为5）和整体（默认为64）。



## 二、Caching



## 三、Connections





## 四、Events



## 五、HTTPS





## 六、Interceptor



## 七、Recipes



## 八、Security











































---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)