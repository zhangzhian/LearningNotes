# OkHttp：源码详解之桥拦截器（三）

> 源码基于okhttp3 java版本：3.14.x

 **BridgeInterceptor** ，意为 **桥拦截器**，相当于 在 请求发起端 和 网络执行端 架起一座桥，把应用层发出的请求 变为 网络层认识的请求，把网络层执行后的响应 变为 应用层便于应用层使用的结果。 看代码：

```java
public final class BridgeInterceptor implements Interceptor {

  //Cookie管理器，初始化OkhttpClient时创建的
  //默认是CookieJar.NO_COOKIES
  private final CookieJar cookieJar;

  public BridgeInterceptor(CookieJar cookieJar) {
    this.cookieJar = cookieJar;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    // 处理封装"Content-Type", "Content-Length","Transfer-Encoding","Host","Connection",
    // "Accept-Encoding","Cookie","User-Agent"等请求头
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }
    // 在服务器支持gzip压缩的前提下，客户端不设置Accept-Encoding=gzip的话，
    // okhttp会自动帮我们开启gzip和解压数据，如果客户端自己开启了gzip，就需要自己解压服务器返回的数据了。
    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }

    //从cookieJar中获取cookie，添加到header
    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }
    // 把处理好的新请求往下传递，执行后续的拦截器的逻辑
    Response networkResponse = chain.proceed(requestBuilder.build());
    //从networkResponse中获取 header "Set-Cookie" 存入cookieJar
    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());
    // 获取返回体的Builder
    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);
    // 处理返回的Response的"Content-Encoding"、"Content-Length"、"Content-Type"等返回头
    // 如果我们没有手动添加"Accept-Encoding: gzip"，这里会创建 能自动解压的responseBody--GzipSource
    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }
    //然后新构建的response返回出去
    return responseBuilder.build();
  }

  /** Returns a 'Cookie' HTTP request header with all cookies, like {@code a=b; c=d}. */
  private String cookieHeader(List<Cookie> cookies) {
    StringBuilder cookieHeader = new StringBuilder();
    for (int i = 0, size = cookies.size(); i < size; i++) {
      if (i > 0) {
        cookieHeader.append("; ");
      }
      Cookie cookie = cookies.get(i);
      cookieHeader.append(cookie.name()).append('=').append(cookie.value());
    }
    return cookieHeader.toString();
  }
}

```

首先，chain.proceed() 执行前，对 **请求添加了header**：“Content-Type”、“Content-Length” 或 “Transfer-Encoding”、“Host”、“Connection”、“Accept-Encoding”、“Cookie”、“User-Agent”，即网络层真正可执行的请求。其中，**注意到，默认是没有cookie处理的，需要我们在初始化OkhttpClient时配置我们自己的cookieJar**。

chain.proceed() 执行后，先把响应header中的cookie存入cookieJar（如果有），然后如果没有手动添加请求header “Accept-Encoding: gzip”，那么会通过 创建能自动解压的responseBody——GzipSource，接着构建新的response返回。





---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)