# OkHttp：源码详解之请求服务拦截器（六）

> 源码基于okhttp3 java版本：3.14.x

请求服务拦截器，也就是真正地去进行网络IO读写了——写入http请求的header和body数据、读取响应的header和body。

ConnectInterceptor主要介绍了如何寻找连接以及连接池如何管理连接。在获取到连接后，调用了RealConnection的newCodec方法ExchangeCodec实例，然后使用ExchangeCodec实例创建了Exchange实例传入CallServerInterceptor了。

ExchangeCodec负责请求和响应的IO读写。

**ExchangeCodec创建过程**——RealConnection的newCodec方法：

```java
  ExchangeCodec newCodec(OkHttpClient client, Interceptor.Chain chain) throws SocketException {
    // http2Connection不为空就创建Http2ExchangeCodec，否则是Http1ExchangeCodec
    if (http2Connection != null) {
      return new Http2ExchangeCodec(client, this, chain, http2Connection);
    } else {
      socket.setSoTimeout(chain.readTimeoutMillis());
      source.timeout().timeout(chain.readTimeoutMillis(), MILLISECONDS);
      sink.timeout().timeout(chain.writeTimeoutMillis(), MILLISECONDS);
      return new Http1ExchangeCodec(client, this, source, sink);
    }
  }
```

http2Connection不为空就创建Http2ExchangeCodec，否则是Http1ExchangeCodec。而http2Connection的创建是连接进行TCP、TLS握手的时候，即在RealConnection的connect方法中，具体就是connect方法中调用的establishProtocol方法。

connect方法：

```java
  public void connect(int connectTimeout, int readTimeout, int writeTimeout,
      int pingIntervalMillis, boolean connectionRetryEnabled, Call call,
      EventListener eventListener) {
    if (protocol != null) throw new IllegalStateException("already connected");

    RouteException routeException = null;
    List<ConnectionSpec> connectionSpecs = route.address().connectionSpecs();
    ConnectionSpecSelector connectionSpecSelector = new ConnectionSpecSelector(connectionSpecs);

    if (route.address().sslSocketFactory() == null) {
      if (!connectionSpecs.contains(ConnectionSpec.CLEARTEXT)) {
        throw new RouteException(new UnknownServiceException(
            "CLEARTEXT communication not enabled for client"));
      }
      String host = route.address().url().host();
      if (!Platform.get().isCleartextTrafficPermitted(host)) {
        throw new RouteException(new UnknownServiceException(
            "CLEARTEXT communication to " + host + " not permitted by network security policy"));
      }
    } else {
      if (route.address().protocols().contains(Protocol.H2_PRIOR_KNOWLEDGE)) {
        throw new RouteException(new UnknownServiceException(
            "H2_PRIOR_KNOWLEDGE cannot be used with HTTPS"));
      }
    }

    while (true) {
      try {
        // 如果是HTTP的代理隧道，那么就会走代理隧道的加载逻辑；
        if (route.requiresTunnel()) {
          connectTunnel(connectTimeout, readTimeout, writeTimeout, call, eventListener);
          if (rawSocket == null) {
            // We were unable to connect the tunnel but properly closed down our resources.
            break;
          }
        } else {
          // 走正常的连接逻辑，无代理
          // 且通过okio创建socket的输入输出流，CallServerInterceptor要用到
          connectSocket(connectTimeout, readTimeout, call, eventListener);
        }
        // 建立协议，这里会触发TLS的握手
        establishProtocol(connectionSpecSelector, pingIntervalMillis, call, eventListener);
        eventListener.connectEnd(call, route.socketAddress(), route.proxy(), protocol);
        break;
      } catch (IOException e) {
        closeQuietly(socket);
        closeQuietly(rawSocket);
        socket = null;
        rawSocket = null;
        source = null;
        sink = null;
        handshake = null;
        protocol = null;
        http2Connection = null;

        eventListener.connectFailed(call, route.socketAddress(), route.proxy(), null, e);

        if (routeException == null) {
          routeException = new RouteException(e);
        } else {
          routeException.addConnectException(e);
        }

        if (!connectionRetryEnabled || !connectionSpecSelector.connectionFailed(e)) {
          throw routeException;
        }
      }
    }

    if (route.requiresTunnel() && rawSocket == null) {
      ProtocolException exception = new ProtocolException("Too many tunnel connections attempted: "
          + MAX_TUNNEL_ATTEMPTS);
      throw new RouteException(exception);
    }

    if (http2Connection != null) {
      synchronized (connectionPool) {
        allocationLimit = http2Connection.maxConcurrentStreams();
      }
    }
  }
```

establishProtocol方法：

```java
  private void establishProtocol(ConnectionSpecSelector connectionSpecSelector,
      int pingIntervalMillis, Call call, EventListener eventListener) throws IOException {
    //针对http请求，如果配置的协议包含Protocol.H2_PRIOR_KNOWLEDGE，则开启Http2连接
    if (route.address().sslSocketFactory() == null) {
      if (route.address().protocols().contains(Protocol.H2_PRIOR_KNOWLEDGE)) {
        socket = rawSocket;
        protocol = Protocol.H2_PRIOR_KNOWLEDGE;
        startHttp2(pingIntervalMillis);
        return;
      }

      socket = rawSocket;
      protocol = Protocol.HTTP_1_1;
      return;
    }
    //针对https请求，会在TLS握手后，根据平台获取协议，如果协议是Protocol.HTTP_2，则开启Http2连接
    eventListener.secureConnectStart(call);
    connectTls(connectionSpecSelector);
    eventListener.secureConnectEnd(call, handshake);

    if (protocol == Protocol.HTTP_2) {
      startHttp2(pingIntervalMillis);
    }
  }
```

好了，到这里不再深入了，继续了解可以参考[HTTP 2.0与OkHttp](https://blog.csdn.net/lmh_19941113/article/details/87896121)。那么到这里，ExchangeCodec已经创建了，然后又包装成Exchange，最后传入了CallServerInterceptor。

下面就来看看这最后一个拦截器CallServerInterceptor ：

```java
public final class CallServerInterceptor implements Interceptor {
  private final boolean forWebSocket;

  public CallServerInterceptor(boolean forWebSocket) {
    this.forWebSocket = forWebSocket;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    //ConnectInterceptor拦截器传入的exchange
    Exchange exchange = realChain.exchange();
    Request request = realChain.request();

    long sentRequestMillis = System.currentTimeMillis();

    // 将请求头写入到socket中，底层通过ExchangeCodec协议类
    // （对应Http1ExchangeCodec和Http2ExchangeCodec）
    // 最终是通过Okio来实现的，具体实现在RealBufferedSink这个类里面
    exchange.writeRequestHeaders(request);

    // 如果有body的话，通过Okio将body写入到socket中，用于发送给服务器
    boolean responseHeadersStarted = false;
    Response.Builder responseBuilder = null;
    //含body的请求
    if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
      // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
      // Continue" response before transmitting the request body. If we don't get that, return
      // what we did get (such as a 4xx response) without ever transmitting the request body.
      // 若请求头包含 "Expect: 100-continue" ,
      // 就会等服务端返回含有 "HTTP/1.1 100 Continue"的响应，然后再发送请求body.
      // 如果没有收到这个响应（例如收到的响应是4xx），那就不发送body了。
      if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
        exchange.flushRequest();
        responseHeadersStarted = true;
        exchange.responseHeadersStart();
        //读取响应头
        responseBuilder = exchange.readResponseHeaders(true);
      }
      // responseBuilder为null说明服务端返回了100，也就是可以继续发送body了
      // 底层通过ExchangeCodec协议类（对应Http1ExchangeCodec和Http2ExchangeCodec）来读取返回头header的数据
      if (responseBuilder == null) {
        if (request.body().isDuplex()) {//默认是false不会进入
          // Prepare a duplex body so that the application can send a request body later.
          exchange.flushRequest();
          BufferedSink bufferedRequestBody = Okio.buffer(
              exchange.createRequestBody(request, true));
          request.body().writeTo(bufferedRequestBody);
        } else {
          // Write the request body if the "Expect: 100-continue" expectation was met.
          // 满足了 "Expect: 100-continue" ，写请求body
          BufferedSink bufferedRequestBody = Okio.buffer(
              exchange.createRequestBody(request, false));
          request.body().writeTo(bufferedRequestBody);
          bufferedRequestBody.close();
        }
      } else {
        //没有满足 "Expect: 100-continue" ，请求发送结束
        exchange.noRequestBody();
        if (!exchange.connection().isMultiplexed()) {
          // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection
          // from being reused. Otherwise we're still obligated to transmit the request body to
          // leave the connection in a consistent state.
          exchange.noNewExchangesOnConnection();
        }
      }
    } else {
      //没有body，请求发送结束
      exchange.noRequestBody();
    }

    //请求发送结束
    if (request.body() == null || !request.body().isDuplex()) {
      //真正将写到socket输出流的http请求数据发送。
      exchange.finishRequest();
    }
    //回调 读响应头开始事件（如果上面没有）
    if (!responseHeadersStarted) {
      exchange.responseHeadersStart();
    }
    //读响应头（如果上面没有）
    if (responseBuilder == null) {
      responseBuilder = exchange.readResponseHeaders(false);
    }

    // 创建返回体Response
    Response response = responseBuilder
        .request(request)
        .handshake(exchange.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();

    int code = response.code();
    if (code == 100) {
      // server sent a 100-continue even though we did not request one.
      // try again to read the actual response
      // 服务端又返回了个100，就再尝试获取真正的响应
      response = exchange.readResponseHeaders(false)
          .request(request)
          .handshake(exchange.connection().handshake())
          .sentRequestAtMillis(sentRequestMillis)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();

      code = response.code();
    }
    //回调读响应头结束
    exchange.responseHeadersEnd(response);
    //这里就是获取响应body了
    if (forWebSocket && code == 101) {
      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
      response = response.newBuilder()
          .body(Util.EMPTY_RESPONSE)
          .build();
    } else {
      // 读取返回体body的数据
      // 底层通过ExchangeCodec协议类（对应Http1ExchangeCodec和Http2ExchangeCodec）
      response = response.newBuilder()
          .body(exchange.openResponseBody(response))
          .build();
    }
    //请求头中Connection是close，表示请求完成后要关闭连接
    if ("close".equalsIgnoreCase(response.request().header("Connection"))
        || "close".equalsIgnoreCase(response.header("Connection"))) {
      exchange.noNewExchangesOnConnection();
    }
    //204（无内容）、205（重置内容），body应该是空
    if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
      throw new ProtocolException(
          "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
    }

    return response;
  }
}

```

写入http请求的header和body、读取响应的header和body。

无论写请求还是读响应，都是使用Exchange对应的方法。上面也提到过Exchange理解上是对ExchangeCodec的包装，这写方法内部除了事件回调和一些参数获取外，核心工作都由 ExchangeCodec 对象完成，而 ExchangeCodec实际上利用的是 Okio，而 Okio 实际上还是用的 Socket。

ExchangeCodec的实现类 有对应Http1.1的Http1ExchangeCodec 和 对应Http2.0的Http2ExchangeCodec。其中Http2ExchangeCodec是使用Http2.0中 **数据帧** 的概念完成请求响应的读写。关于Http1ExchangeCodec、Http2ExchangeCodec具体实现原理涉及okio这不再展开。

最后一点，CallServerInterceptor的intercept方法中没有调用连接器链Chain的proceed方法，这是最后一个拦截器。

---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)