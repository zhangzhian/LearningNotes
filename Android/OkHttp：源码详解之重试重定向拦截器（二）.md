# OkHttp：源码详解之重试重定向拦截器（二）

> 源码基于okhttp3 java版本：3.14.x

上一篇《OkHttp：源码详解之核心流程（一）》文章详细的描述了OkHttp发起一个请求的整体流程，详细读者已经对整体流程有一个较为清晰的认知。接下来我们开始依次分析5个系统添加的拦截器，通过这5个拦截器的分析，掌握OkHttp是如何进行一次真正的网络请求。

如果请求创建时没有添加应用拦截器，那么第一个拦截器就是**RetryAndFollowUpInterceptor**，意为“**重试和重定向拦截器**”，作用是**连接失败后进行重试、对请求结果跟进后进行重定向**。

通过前面文章的分析，我们知道它的执行逻辑在intercept方法中：

```java
 @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Transmitter transmitter = realChain.transmitter();

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      // 准备连接请求
      // 主机、端口、协议都相同时，连接可复用
      transmitter.prepareToConnect(request);

      if (transmitter.isCanceled()) {
        throw new IOException("Canceled");
      }

      Response response;
      boolean success = false;
      try {
        // 执行其他(下一个->下一个->...)拦截器的功能，获取Response；
        response = realChain.proceed(request, transmitter, null);
        success = true;
      } catch (RouteException e) {
        // 连接路由异常，此时请求还未发送。尝试恢复
        // 返回true，continue，继续下一个while循环
        // The attempt to connect via a route failed. The request will not have been sent.
        if (!recover(e.getLastConnectException(), transmitter, false, request)) {
          throw e.getFirstConnectException();
        }
        continue;
      } catch (IOException e) {
        // IO异常，请求可能已经发出。尝试恢复
        // An attempt to communicate with a server failed. The request may have been sent.
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, transmitter, requestSendStarted, request)) throw e;
        continue;
      } finally {
        // 请求没成功，释放资源
        // The network call threw an exception. Release any resources.
        if (!success) {
          transmitter.exchangeDoneDueToException();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }

      Exchange exchange = Internal.instance.exchange(response);
      Route route = exchange != null ? exchange.connection().route() : null;
      // 后面的拦截器执行完了，拿到Response
      // 解析看下是否需要重试或重定向，需要则返回新的Request
      Request followUp = followUpRequest(response, route);

      if (followUp == null) {
        if (exchange != null && exchange.isDuplex()) {
          transmitter.timeoutEarlyExit();
        }
        // 如果followUpRequest返回的Request为空，那边就表示不需要执行重试或者重定向，直接返回数据
        return response;
      }

      RequestBody followUpBody = followUp.body();
      if (followUpBody != null && followUpBody.isOneShot()) {
        // 如果followUp不为null，请求体不为空，但只需要请求一次时，那么就返回response；
        return response;
      }

      closeQuietly(response.body());
      if (transmitter.hasExchange()) {
        exchange.detachWithViolence();
      }

      // 判断重试或者重定向的次数是否超过最大的次数20，是的话则抛出异常
      if (++followUpCount > MAX_FOLLOW_UPS) {
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }
      // 将需要重试或者重定向的请求赋值给新的请求
      // 继续执行新请求
      request = followUp;
      priorResponse = response;
    }//...while (true)
  }
```

我们注意到主要执行逻辑是使用**while进行循环**的，循环的意义在于，如果是重试或重定向再次执行这一过程，否则的话，会通过return 或是throw跳出循环。

先用transmitter.prepareToConnect(request)进行连接准备。transmitter意为发射器，是应用层和网络层的桥梁，在进行 连接、真正发出请求和读取响应中起到很重要的作用。看下prepareToConnect方法：

```java
public void prepareToConnect(Request request) {
    if (this.request != null) {
      if (sameConnection(this.request.url(), request.url()) && exchangeFinder.hasRouteToTry()) {
        return; // Already ready.//已有相同连接
      }
      if (exchange != null) throw new IllegalStateException();

      if (exchangeFinder != null) {
        maybeReleaseConnection(null, true);
        exchangeFinder = null;
      }
    }

    this.request = request;
    //创建ExchangeFinder，目的是为获取连接做准备
    //ExchangeFinder是交换查找器，作用是获取请求的连接
    //connectionPool是在OkHttpClient.Build->new ConnectionPool->new RealConnectionPool创建的
    //createAddress方法返回的Address
    //注意:构造函数中创建了RouteSelector
    this.exchangeFinder = new ExchangeFinder(this, connectionPool, createAddress(request.url()),
        call, eventListener);
  }
```

主要是创建ExchangeFinder赋值给transmitter.exchangeFinder。ExchangeFinder是交换查找器，作用是获取请求的连接。这里先了解下，后面会具体说明。

接着调用realChain.proceed 继续传递请求给下一个拦截器、从下一个拦截器获取原始结果。如果此过程发生了 连接路由异常 或 IO异常，就会调用recover判断是否进行重试恢复。看下recover方法：

```java
  private boolean recover(IOException e, Transmitter transmitter,
      boolean requestSendStarted, Request userRequest) {

    // 应用层禁止重试，就不重试
    // The application layer has forbidden retries.
    if (!client.retryOnConnectionFailure()) return false;

    // 不能再次发送请求，就不重试
    // We can't send the request body again.
    if (requestSendStarted && requestIsOneShot(e, userRequest)) return false;

    // 发生的异常是致命的，就不重试
    // This exception is fatal.
    if (!isRecoverable(e, requestSendStarted)) return false;

    // 没有路由可以尝试，就不重试
    // No more routes to attempt.
    if (!transmitter.canRetry()) return false;

    // For failure recovery, use the same route selector with a new connection.
    return true;
  }
```

如果recover方法返回true，那么就会进入下一次循环，重新请求。

如果realChain.proceed没有发生异常，返回了结果response，使用**followUpRequest**方法跟进结果并构建重定向request。 如果不用跟进处理（例如响应码是200），则返回null。看下followUpRequest方法：

```java
 private Request followUpRequest(Response userResponse, @Nullable Route route) throws IOException {
    if (userResponse == null) throw new IllegalStateException();
    int responseCode = userResponse.code();

    final String method = userResponse.request().method();
    //根据返回码进行不同的操作
    switch (responseCode) {
      case HTTP_PROXY_AUTH:
        Proxy selectedProxy = route != null
            ? route.proxy()
            : client.proxy();
        if (selectedProxy.type() != Proxy.Type.HTTP) {
          throw new ProtocolException("Received HTTP_PROXY_AUTH (407) code while not using proxy");
        }
        return client.proxyAuthenticator().authenticate(route, userResponse);

      case HTTP_UNAUTHORIZED:
        return client.authenticator().authenticate(route, userResponse);

      case HTTP_PERM_REDIRECT:
      case HTTP_TEMP_REDIRECT:
        // "If the 307 or 308 status code is received in response to a request other than GET
        // or HEAD, the user agent MUST NOT automatically redirect the request"
        if (!method.equals("GET") && !method.equals("HEAD")) {
          return null;
        }
        // fall-through
      case HTTP_MULT_CHOICE:
      case HTTP_MOVED_PERM:
      case HTTP_MOVED_TEMP:
      case HTTP_SEE_OTHER:
        // Does the client allow redirects?
        if (!client.followRedirects()) return null;

        String location = userResponse.header("Location");
        if (location == null) return null;
        HttpUrl url = userResponse.request().url().resolve(location);

        // Don't follow redirects to unsupported protocols.
        if (url == null) return null;

        // If configured, don't follow redirects between SSL and non-SSL.
        boolean sameScheme = url.scheme().equals(userResponse.request().url().scheme());
        if (!sameScheme && !client.followSslRedirects()) return null;

        // Most redirects don't include a request body.
        Request.Builder requestBuilder = userResponse.request().newBuilder();
        if (HttpMethod.permitsRequestBody(method)) {
          final boolean maintainBody = HttpMethod.redirectsWithBody(method);
          if (HttpMethod.redirectsToGet(method)) {
            requestBuilder.method("GET", null);
          } else {
            RequestBody requestBody = maintainBody ? userResponse.request().body() : null;
            requestBuilder.method(method, requestBody);
          }
          if (!maintainBody) {
            requestBuilder.removeHeader("Transfer-Encoding");
            requestBuilder.removeHeader("Content-Length");
            requestBuilder.removeHeader("Content-Type");
          }
        }

        // When redirecting across hosts, drop all authentication headers. This
        // is potentially annoying to the application layer since they have no
        // way to retain them.
        if (!sameConnection(userResponse.request().url(), url)) {
          requestBuilder.removeHeader("Authorization");
        }

        return requestBuilder.url(url).build();

      case HTTP_CLIENT_TIMEOUT:
      ...
 }
```

主要就是根据响应码判断如果需要重定向，就从响应中取出重定向的url并构建新的Request并返回出去。

最下面还有个判断：最多重试20次。

综上，RetryAndFollowUpInterceptor主要就是连接失败的重试、跟进处理重定向。





---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)