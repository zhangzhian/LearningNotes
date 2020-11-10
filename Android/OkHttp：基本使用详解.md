# OkHttp：基本使用详解
## 简介
OkHttp是一个高效的HTTP客户端，它有以下默认特性：

- 支持HTTP/2，允许所有同一个主机地址的请求共享同一个socket连接
- 连接池减少请求延时
- 透明的GZIP压缩减少响应数据的大小
- 缓存响应内容，避免一些完全重复的请求

当网络出现问题的时候OkHttp依然坚守自己的职责，它会自动恢复一般的连接问题，如果你的服务有多个IP地址，当第一个IP请求失败时，OkHttp会交替尝试你配置的其他IP，这对于使用IPv4+IPv6托管在冗余数据中心中的服务是必需的，OkHttp使用现代TLS技术(TLS 1.3、ALPN、证书固定)初始化新的连接，当握手失败时会回退到TLS 1.0。

 OkHttp官网地址：http://square.github.io/okhttp/ 
 OkHttp GitHub地址：https://github.com/square/okhttp 

本文代码下载地址：[GitHub](https://github.com/zhangzhian/StarDust/tree/master/app/src/main/java/com/zza/stardust/app/ui/OkHttp)


## 使用

```c
implementation 'com.squareup.okhttp3:okhttp:3.14.2'
```

我使用的是3.14.2版本，4.XX.XX版本使用的是Kotlin

需要在清单文件声明访问Internet的权限，如果使用缓存，那还得声明写外存的权限

### 异步Get请求

```java
   /**
     * 异步GET请求:
     * new OkHttpClient;
     * 构造Request对象；
     * 通过前两步中的对象构建Call对象；
     * 通过Call#enqueue(Callback)方法来提交异步请求；
     */
    private void asynchronousGetRequests() {
        String url = "https://wwww.baidu.com";
        OkHttpClient okHttpClient = new OkHttpClient();
        final Request request = new Request.Builder()
                .url(url)
                .get()//默认就是GET请求
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("asynchronousGetRequests onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String result = response.body().string();
                runOnUiThread(() -> tvContent.setText("asynchronousGetRequests onResponse: " + result));

            }
        });
    }
```
异步发起的请求会被加入到 `Dispatcher` 中的 `runningAsyncCalls`双端队列中通过线程池来执行。

> 响应体的 string() 方法对于小文档来说十分方便、高效。但是如果响应体太大（超过1MB），应避免适应 string()方法，因为他会将把整个文档加载到内存中。对于超过1MB的响应body，应使用流的方式来处理body。

### 同步Get请求

在Android中应放在子线程中执行，否则有可能引起ANR异常，Android3.0 以后已经不允许在主线程访问网络。

```java
    /**
     * 同步GET请求
     * new OkHttpClient;
     * 构造Request对象；
     * 通过前两步中的对象构建Call对象；
     * 在子线程中通过Call#execute()方法来提交同步请求；
     */
    private void synchronizedGetRequests() {
        String url = "https://wwww.baidu.com";
        OkHttpClient okHttpClient = new OkHttpClient();
        final Request request = new Request.Builder()
                .url(url)
                .get()//默认就是GET请求
                .build();
        final Call call = okHttpClient.newCall(request);
        new Thread(() -> {
            try {
                //直接execute call
                Response response = call.execute();
                String result = response.body().string();
                runOnUiThread(() -> tvContent.setText("synchronizedGetRequests run: " + result));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
```

### POST方式提交String

RequstBody的几种构造方式：

![RequstBody的几种构造方式：](https://upload-images.jianshu.io/upload_images/3631399-304a944b5bef663e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/921)



```java
    /**
     * POST方式提交String
     * 在构造 Request对象时，需要多构造一个RequestBody对象，携带要提交的数据。
     * 在构造 RequestBody 需要指定MediaType，用于描述请求/响应 body 的内容类型
     */
    private void postString() {
        MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
        String requestBody = "I am zza.";
        Request request = new Request.Builder()
                .url("https://api.github.com/markdown/raw")
                .post(RequestBody.create(mediaType, requestBody))
                .build();
        OkHttpClient okHttpClient = new OkHttpClient();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("postString onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                StringBuffer buffer = new StringBuffer();
                buffer.append("postString: " + "\r\n");
                buffer.append(response.protocol() + " " + response.code() + " " + response.message() + "\r\n");
                Headers headers = response.headers();
                for (int i = 0; i < headers.size(); i++) {
                    buffer.append(headers.name(i) + ":" + headers.value(i) + "\r\n");
                }
                buffer.append("onResponse: " + response.body().string());

                runOnUiThread(() -> tvContent.setText(buffer.toString()));
            }
        });
    }
```

### POST方式提交流

```java
    /**
     * POST方式提交流
     */
    private void postStream() {
        RequestBody requestBody = new RequestBody() {
            @Nullable
            @Override
            public MediaType contentType() {
                return MediaType.parse("text/x-markdown; charset=utf-8");
            }

            @Override
            public void writeTo(BufferedSink sink) throws IOException {
                sink.writeUtf8("I am zza.");
            }
        };

        Request request = new Request.Builder()
                .url("https://api.github.com/markdown/raw")
                .post(requestBody)
                .build();
        OkHttpClient okHttpClient = new OkHttpClient();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("postStream onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                StringBuffer buffer = new StringBuffer();
                buffer.append("postStream: " + "\r\n");
                buffer.append(response.protocol() + " " + response.code() + " " + response.message() + "\r\n");
                Headers headers = response.headers();
                for (int i = 0; i < headers.size(); i++) {
                    buffer.append(headers.name(i) + ":" + headers.value(i) + "\r\n");
                }
                buffer.append("onResponse: " + response.body().string());

                runOnUiThread(() -> tvContent.setText(buffer.toString()));
            }
        });
    }
```


### POST提交文件

```java
    /**
     * POST提交文件
     * 文件没有的话会失败
     * 需要在路径下添加文件
     * 还需要权限
     */
    private void postFile() {
        MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
        OkHttpClient okHttpClient = new OkHttpClient();
        String path = Environment.getExternalStorageDirectory().getPath() + "/zza.md";
        File file = new File(path);
        Request request = new Request.Builder()
                .url("https://api.github.com/markdown/raw")
                .post(RequestBody.create(mediaType, file))
                .build();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("postFile onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                StringBuffer buffer = new StringBuffer();
                buffer.append("postFile: " + "\r\n");
                buffer.append(response.protocol() + " " + response.code() + " " + response.message() + "\r\n");
                Headers headers = response.headers();
                for (int i = 0; i < headers.size(); i++) {
                    buffer.append(headers.name(i) + ":" + headers.value(i) + "\r\n");
                }
                buffer.append("postFile: " + response.body().string());

                runOnUiThread(() -> tvContent.setText(buffer.toString()));
            }
        });
    }
```

### POST方式提交表单

提交表单时，使用 `RequestBody` 的实现类`FormBody`来描述请求体，它可以携带一些经过编码的 `key-value` 请求体，键值对存储在下面两个集合中：

```java
  private final List<String> encodedNames;
  private final List<String> encodedValues;
```



```java
   /**
     * POST方式提交表单
     * 通过FormBody#Builder构造RequestBody
     */
    private void postForm() {
        //https://www.wanandroid.com/article/query/0/json
        //方法：POST
        //参数：
        //  页码：拼接在链接上，从0开始。
        //  k ： 搜索关键词
        RequestBody requestBody = new FormBody.Builder()
                .add("k", "okhttp")
                .build();
        Request request = new Request.Builder()
                .url("https://www.wanandroid.com/article/query/0/json")
                .post(requestBody)
                .build();
        OkHttpClient okHttpClient = new OkHttpClient();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("postForm onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                StringBuffer buffer = new StringBuffer();
                buffer.append("postForm: " + "\r\n");
                buffer.append(response.protocol() + " " + response.code() + " " + response.message() + "\r\n");
                Headers headers = response.headers();
                for (int i = 0; i < headers.size(); i++) {
                    buffer.append(headers.name(i) + ":" + headers.value(i) + "\r\n");
                }
                buffer.append("postForm: " + response.body().string());

                runOnUiThread(() -> tvContent.setText(buffer.toString()));
            }
        });
    }
```

### POST方式提交分块请求

MultipartBody 可以构建复杂的请求体，与HTML文件上传形式兼容。多块请求体中每块请求都是一个请求体，可以定义自己的请求头。这些请求头可以用来描述这块请求，例如它的 `Content-Disposition`。如果 `Content-Length`和 `Content-Type`可用的话，他们会被自动添加到请求头中。

```java
    /**
     * POST方式提交分块请求
     * <p>
     * 使用公司项目的一个接口调试通过，为避免一些问题，接口和参数删掉
     */
    private void postMultipartBody() {
        OkHttpClient client = new OkHttpClient();

        File file = new File(Environment.getExternalStorageDirectory().getPath() + "/logo_star_dust.png");
        MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");
        RequestBody filebody = MultipartBody.create(MEDIA_TYPE_PNG, file);

        MultipartBody body = new MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart("data", file.getName(), filebody)
                .addPart(
                        Headers.of("Content-Disposition", "form-data; name=\"vin\""),
                        RequestBody.create(null, ""))
                .addPart(
                        Headers.of("Content-Disposition", "form-data; name=\"iccid\""),
                        RequestBody.create(null, ""))
                .addPart(
                        Headers.of("Content-Disposition", "form-data; name=\"type\""),
                        RequestBody.create(null, ""))
                .addPart(
                        Headers.of("Content-Disposition", "form-data; name=\"jobId\""),
                        RequestBody.create(null, ""))
                .build();

        Request request = new Request.Builder()
                .url("https://www.baidu.com")
                .post(body)
                .build();
        Call call = client.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("postMultipartBody onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String result = response.body().string();
                runOnUiThread(() -> tvContent.setText("postMultipartBody onResponse: " + result));
            }

        });
    }
```

### 拦截器

OkHttp的拦截器链可谓是其整个框架的精髓，用户可传入的 `interceptor`分为两类：
- 应用拦截器(Interceptors)
	- `addInterceptor()` 
	- 不必要担心响应和重定向之间的中间响应。
	- 通常只调用一次，即使HTTP响应是通过缓存提供的。
	- 遵从应用层的最初目的。与OkHttp的注入头部无关，如If-None-Match。
	- 允许短路而且不调用Chain.proceed()。
	- 允许重试和多次调用Chain.proceed()。

- 网络拦截器(networkInterceptors)
	- `addNetworkInterceptor()` 
	- 允许像重定向和重试一样操作中间响应。
	- 网络发生短路时不调用缓存响应。
	- 在数据被传递到网络时观察数据。
	- 有权获得装载请求的连接。

```java
    /**
     * 拦截器
     * <p>
     * 请查看打印输出
     */
    private void testInterceptor() {
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .addInterceptor(new LoggingInterceptor())
                .build();
        Request request = new Request.Builder()
                .url("https://www.publicobject.com/helloworld.txt")
                .header("User-Agent", "OkHttp Example")
                .build();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("testInterceptor onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                ResponseBody body = response.body();
                String result = response.body().string();
                if (body != null) {
                    LogUtil.d("testInterceptor: " + result);
                    body.close();
                }
                runOnUiThread(() -> tvContent.setText("testInterceptor onResponse: " + result));
            }
        });
    }
```



### 支持缓存

为了缓存响应，你需要一个你可以读写的缓存目录，和缓存大小的限制。这个缓存目录应该是私有的，不信任的程序应不能读取缓存内容。 

一个缓存目录同时拥有多个缓存访问是错误的。大多数程序只需要调用一次new OkHttp()，在第一次调用时配置好缓存，然后其他地方只需要调用这个实例就可以了。否则两个缓存示例互相干扰，破坏响应缓存，而且有可能会导致程序崩溃。 

```java
    /**
     * 支持缓存
     */
    private void responseCaching() {
        String url = "https://publicobject.com/helloworld.txt";
        File cacheDirectory = new File(Environment.getExternalStorageDirectory() + "/cache");
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(cacheDirectory, cacheSize);

        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .cache(cache)
                .build();

        final Request request = new Request.Builder()
                .url(url)
                .get()//默认就是GET请求
                .build();
        final Call call1 = okHttpClient.newCall(request);
        final Call call2 = okHttpClient.newCall(request);
        new Thread(() -> {
            try {
                StringBuffer buffer = new StringBuffer();
                //直接execute call
                Response response1 = call1.execute();
                if (!response1.isSuccessful()) {
                    runOnUiThread(() -> tvContent.setText("responseCaching onFailure"));
                } else {
                    String result = response1.body().string();

                    buffer.append("responseCaching1 onResponse:   " + result + "\r\n");
                    buffer.append("responseCaching1 cache response:    " + response1.cacheResponse() + "\r\n");
                    buffer.append("responseCaching1 network response:  " + response1.networkResponse() + "\r\n");
                    runOnUiThread(() -> tvContent.setText(buffer));
                }
                Response response2 = call2.execute();
                if (!response2.isSuccessful()) {
                    runOnUiThread(() -> tvContent.setText("responseCaching onFailure"));
                } else {
                    String result = response2.body().string();
                    buffer.append("responseCaching2 onResponse:   " + result + "\r\n");
                    buffer.append("responseCaching2 cache response:    " + response2.cacheResponse() + "\r\n");
                    buffer.append("responseCaching2 network response:  " + response2.networkResponse() + "\r\n");
                    runOnUiThread(() -> tvContent.setText(buffer));
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
```

为了防止使用缓存的响应，可以用`CacheControl.FORCE_NETWORK`。为了防止它使用网络，使用`CacheControl.FORCE_CACHE`。需要注意的是：如果您使用FORCE_CACHE和网络的响应需求，OkHttp则会返回一个504提示，告诉你不可满足请求响应。 



### 取消请求



使用Call.cancel()可以立即停止掉一个正在执行的call。如果一个线程正在写请求或者读响应，将会引发IOException。当call没有必要的时候，使用这个api可以节约网络资源。例如当用户离开一个应用时。

不管同步还是异步的call都可以取消。 

你可以通过tags来同时取消多个请求。当你构建一请求时，使用RequestBuilder.tag(tag)来分配一个标签。之后你就可以用OkHttpClient.cancel(tag)来取消所有带有这个tag的call。.

```java
    /**
     * 取消请求
     */
    private void cancelCall() {
        final ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
        final OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url("https://httpbin.org/delay/2") // This URL is served with a 2 second delay.
                .build();

        final long startNanos = System.nanoTime();
        final Call call = client.newCall(request);

        StringBuffer buffer = new StringBuffer();
        // Schedule a job to cancel the call in 1 second.
        executor.schedule(new Runnable() {
            @Override
            public void run() {
                buffer.append(String.format("%.2f Canceling call.%n",
                        (System.nanoTime() - startNanos) / 1e9f));
                call.cancel();
                buffer.append(String.format("%.2f Canceled call.%n",
                        (System.nanoTime() - startNanos) / 1e9f) );
                runOnUiThread(() -> tvContent.setText(buffer));
            }
        }, 1, TimeUnit.SECONDS);

        executor.schedule(new Runnable() {
            @Override
            public void run() {
                buffer.append(String.format("%.2f Executing call.%n", (System.nanoTime() - startNanos) / 1e9f) );
                try (Response response = call.execute()) {
                    buffer.append(String.format("%.2f Call was expected to fail, but completed: %s%n",
                            (System.nanoTime() - startNanos) / 1e9f, response));
                } catch (IOException e) {
                    buffer.append(String.format("%.2f Call failed as expected: %s%n",
                            (System.nanoTime() - startNanos) / 1e9f, e) );
                }
            }
        }, 0, TimeUnit.SECONDS);
    }

```

### 超时

OkHttp支持连接，读取和写入超时。

```java
    /**
     *  超时
     */
    private void timeout() {
        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(10, TimeUnit.SECONDS)
                .writeTimeout(10, TimeUnit.SECONDS)
                .readTimeout(10, TimeUnit.SECONDS)
                .build();

         Request request = new Request.Builder()
                .url("https://httpbin.org/delay/12")
                .get()
                .build();
        final Call call = client.newCall(request);

        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("timeout onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String result = response.body().string();
                StringBuffer buffer = new StringBuffer();
                buffer.append("timeout onResponse:   " + result + "\r\n");
                runOnUiThread(() -> tvContent.setText(buffer));

            }
        });
    }

```

### 每个call配置

使用OkHttpClient，所有的HTTP Client配置包括代理设置、超时设置、缓存设置。当你需要为单个call改变配置的时候，clone 一个 OkHttpClient。这个api将会返回一个浅拷贝（shallow copy），你可以用来单独自定义。

```java
/**
 * 每个call配置
 */
private void perCallConfiguration() {
    OkHttpClient client = new OkHttpClient.Builder()
            .build();

    Request request = new Request.Builder()
            .url("https://httpbin.org/delay/1")
            .get()
            .build();

    OkHttpClient clientCopy = client.newBuilder()
            .readTimeout(500, TimeUnit.MILLISECONDS)
            .build();

    final Call call = clientCopy.newCall(request);
    StringBuffer buffer = new StringBuffer();
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {
            runOnUiThread(() -> tvContent.setText("perCallConfiguration onFailure: " + e.getMessage()));
        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            String result = response.body().string();
            buffer.append("perCallConfiguration onResponse:   " + result + "\r\n");
            runOnUiThread(() -> tvContent.setText(buffer));

        }
    });
}
```
### 处理身份验证

OkHttp会自动重试未验证的请求。当响应是401 Not Authorized时，Authenticator会被要求提供证书。Authenticator的实现中需要建立一个新的包含证书的请求。如果没有证书可用，返回null来跳过尝试。

```java
    /**
     * 处理身份验证
     */
    private void handlingAuthentication() {
        StringBuffer buffer = new StringBuffer();

        OkHttpClient client = new OkHttpClient.Builder()
                .authenticator(new Authenticator() {

                    @Override
                    public Request authenticate(Route route, Response response) throws IOException {
                        if (response.request().header("Authorization") != null) {
                            return null; // Give up, we've already attempted to authenticate.
                        }
                        buffer.append("Authenticating for response: " + response + "\r\n");
                        buffer.append("Challenges: " + response.challenges() + "\r\n");

                        String credential = Credentials.basic("jesse", "password1");
                        return response.request().newBuilder()
                                .header("Authorization", credential)
                                .build();
                    }
                })
                .build();

        Request request = new Request.Builder()
                .url("http://publicobject.com/secrets/hellosecret.txt")
                .get()
                .build();

        final Call call = client.newCall(request);

        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(() -> tvContent.setText("handlingAuthentication onFailure: " + e.getMessage()));
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String result = response.body().string();
                buffer.append("handlingAuthentication onResponse:   " + result + "\r\n");
                runOnUiThread(() -> tvContent.setText(buffer));

            }
        });
    }
```

## 其他

- 推荐让 `OkHttpClient`保持单例，用同一个 `OkHttpClient`实例来执行你的所有请求，因为每一个 `OkHttpClient`实例都拥有自己的连接池和线程池，重用这些资源可以减少延时和节省资源，如果为每个请求创建一个 `OkHttpClient`实例，显然就是一种资源的浪费。
- 每一个Call（其实现是RealCall）只能执行一次，否则会报异常，具体参见 `RealCall#execute()`

---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)