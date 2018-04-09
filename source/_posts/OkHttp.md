
---
title: OkHttp
date: 2018-04-09 23:47:44
categories: "Android三方库"
tags:
     - Android
     - Java
     - 三方框架
---

### 一.基础内容
#### 1.请求体
访问协议, 响应码, 描述信息, 响应头, 响应体

#### 2.使用代码
1.get同步和异步方式
     
   	Request request = new Request.Builder()
        .url("http://publicobject.com/helloworld.txt")
        .build();
 
    Response response = client.newCall(request).execute();
    
	client.newCall(request).enqueue(new Callback() {
      @Override public void onFailure(Request request, Throwable throwable) {
        
      }
 
      @Override public void onResponse(Response response) throws IOException {
        
      }
    });
    

2.post方式提交string

    String postBody = ""
        + "Releases\n"
        + "--------\n"
        + "\n"
        + " * _1.0_ May 6, 2013\n"
        + " * _1.1_ June 15, 2013\n"
        + " * _1.2_ August 11, 2013\n";
 
    Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
        .build();
 
    Response response = client.newCall(request).execute();


3.post方式提交流

以流的方式POST提交请求体. 请求体的内容由流写入产生. 这个例子是流直接写入Okio的BufferedSink. 你的程序可能会使用OutputStream, 你可以使用BufferedSink.outputStream()来获取. OkHttp的底层对流和字节的操作都是基于Okio库, Okio库也是Square开发的另一个IO库, 填补I/O和NIO的空缺, 目的是提供简单便于使用的接口来操作IO.

    RequestBody requestBody = new RequestBody() {
      @Override public MediaType contentType() {
        return MEDIA_TYPE_MARKDOWN;
      }
 
      @Override public void writeTo(BufferedSink sink) throws IOException {
        sink.writeUtf8("Numbers\n");
        sink.writeUtf8("-------\n");
        for (int i = 2; i <= 997; i++) {
          sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
        }
      }
 
      private String factor(int n) {
        for (int i = 2; i < n; i++) {
          int x = n / i;
          if (x * i == n) return factor(x) + " × " + i;
        }
        return Integer.toString(n);
      }
    };
    
4.Post方式提交文件

    File file = new File("README.md");
	RequestBody formBody=RequestBody.create(MEDIA_TYPE_MARKDOWN, file)
    

5.Post方式提交表单

    RequestBody formBody = new FormBody.Builder()
        .add("search", "Jurassic Park")
        .build();

6.Post方式提交分块请求

MultipartBody.Builder可以构建复杂的请求体, 与HTML文件上传形式兼容. 多块请求体中每块请求都是一个请求体, 可以定义自己的请求头. 这些请求头可以用来描述这块请求, 例如它的Content-Disposition. 如果Content-Length和Content-Type可用的话, 他们会被自动添加到请求头中.

    RequestBody requestBody = new MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("title", "Square Logo")
        .addFormDataPart("image", "logo-square.png",
            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
        .build();

    Request request = new Request.Builder()
        .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
        .url("https://api.imgur.com/3/image")
        .post(requestBody)
        .build();

    Response response = client.newCall(request).execute();

7.设置请求头

    Request request = new Request.Builder()
        .url("https://api.github.com/repos/square/okhttp/issues")
        .header("User-Agent", "OkHttp Headers.java")
        .addHeader("Accept", "application/json; q=0.5")
        .addHeader("Accept", "application/vnd.github.v3+json")
        .build();

8.使用Gson来解析JSON响应
	

    private final Gson gson = new Gson();
	Gist gist = gson.fromJson(response.body().charStream(), Gist.class);


 9.响应缓存

大多数程序只需要调用一次new OkHttp(), 在第一次调用时配置好缓存, 然后其他地方只需要调用这个实例就可以了. 否则两个缓存示例互相干扰, 破坏响应缓存, 而且有可能会导致程序崩溃.
响应缓存使用HTTP头作为配置. 你可以在请求头中添加Cache-Control: max-stale=3600 , OkHttp缓存会支持. 你的服务通过响应头确定响应缓存多长时间, 例如使用Cache-Control: max-age=9600.

    int cacheSize = 10 * 1024 * 1024; // 10 MiB
    Cache cache = new Cache(cacheDirectory, cacheSize);
 
    client = new OkHttpClient();
    client.setCache(cache);

10.Force a Network Response or Cache

    connection.addRequestProperty("Cache-Control", "no-cache");
    connection.addRequestProperty("Cache-Control", "only-if-cached");
    InputStream cached = connection.getInputStream();


11.取消一个Call

使用Call.cancel()可以立即停止掉一个正在执行的call. 如果一个线程正在写请求或者读响应, 将会引发IOException. 当call没有必要的时候, 使用这个api可以节约网络资源. 例如当用户离开一个应用时, 不管同步还是异步的call都可以取消.
你可以通过tags来同时取消多个请求. 当你构建一请求时, 使用RequestBuilder.tag(tag)来分配一个标签, 之后你就可以用OkHttpClient.cancel(tag)来取消所有带有这个tag的call.

12.超时

没有响应时使用超时结束call. 没有响应的原因可能是客户点链接问题、服务器可用性问题或者这之间的其他东西. OkHttp支持连接超时, 读取超时和写入超时.

    client = new OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .writeTimeout(10, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .build();

13.每个Call的设置不同

    private final OkHttpClient client = new OkHttpClient();

	public void run() throws Exception {
		Request request = new Request.Builder()
	      .url("http://httpbin.org/delay/1") // This URL is served with a 1 second delay.
	      .build();

	    try {
	      // Copy to customize OkHttp for this request.
	      OkHttpClient copy = client.newBuilder()
	          .readTimeout(500, TimeUnit.MILLISECONDS)
	          .build();
	
	      Response response = copy.newCall(request).execute();
	      System.out.println("Response 1 succeeded: " + response);
	    } catch (IOException e) {
	      System.out.println("Response 1 failed: " + e);
	    }
	
	    try {
	      // Copy to customize OkHttp for this request.
	      OkHttpClient copy = client.newBuilder()
	          .readTimeout(3000, TimeUnit.MILLISECONDS)
	          .build();
	
	      Response response = copy.newCall(request).execute();
	      System.out.println("Response 2 succeeded: " + response);
	    } catch (IOException e) {
	      System.out.println("Response 2 failed: " + e);
        }
    }

### 二.源码解析

 1.OkHttpClient的内容

    public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory {
		public OkHttpClient() {
	       this(new Builder());
		}
		OkHttpClient(Builder builder) {
	    this.dispatcher = builder.dispatcher;
	    this.proxy = builder.proxy;
	    this.protocols = builder.protocols;
	    this.connectionSpecs = builder.connectionSpecs;
	    this.interceptors = Util.immutableList(builder.interceptors);
	    this.networkInterceptors = Util.immutableList(builder.networkInterceptors);
	    this.eventListenerFactory = builder.eventListenerFactory;
	    this.proxySelector = builder.proxySelector;
	    this.cookieJar = builder.cookieJar;
	    this.cache = builder.cache;
	    this.internalCache = builder.internalCache;
	    this.socketFactory = builder.socketFactory;
	
	    boolean isTLS = false;
	    ......
	
	    this.hostnameVerifier = builder.hostnameVerifier;
	    this.certificatePinner = builder.certificatePinner.withCertificateChainCleaner(
	        certificateChainCleaner);
	    this.proxyAuthenticator = builder.proxyAuthenticator;
	    this.authenticator = builder.authenticator;
	    this.connectionPool = builder.connectionPool;
	    this.dns = builder.dns;
	    this.followSslRedirects = builder.followSslRedirects;
	    this.followRedirects = builder.followRedirects;
	    this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
	    this.connectTimeout = builder.connectTimeout;
	    this.readTimeout = builder.readTimeout;
	    this.writeTimeout = builder.writeTimeout;
	    this.pingInterval = builder.pingInterval;
	  }
	}

2.Request内容
		
	Request request = new Request.Builder().url("url").build();


    public final class Request {
	    public Builder() {
	      this.method = "GET";
	      this.headers = new Headers.Builder();
	    }
	    public Builder url(String url) {
	      ......
	
	      // Silently replace web socket URLs with HTTP URLs.
	      if (url.regionMatches(true, 0, "ws:", 0, 3)) {
	        url = "http:" + url.substring(3);
	      } else if (url.regionMatches(true, 0, "wss:", 0, 4)) {
	        url = "https:" + url.substring(4);
	      }
	
	      HttpUrl parsed = HttpUrl.parse(url);
	      ......
	      return url(parsed);
	    }
	    public Request build() {
	      ......
	      return new Request(this);
	    }
	}

3.newCall( Request)

    public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory {
	   @Override 
	   public Call newCall(Request request) {
	    return new RealCall(this, request, false /* for web socket */);
	   }
	}

RealCall实现了Call.Factory接口创建了一个RealCall的实例，而RealCall是Call接口的实现。

4.异步请求

    final class RealCall implements Call {
	   @Override 
	   public void enqueue(Callback responseCallback) {
	   synchronized (this) {
	   if (executed) throw new IllegalStateException("Already Executed");
	      executed = true;
	   }
	    captureCallStackTrace();
	    client.dispatcher().enqueue(new AsyncCall(responseCallback));
	  }
	}

由以上源码得知：

1） 检查这个 call 是否已经被执行了，每个 call 只能被执行一次，如果想要一个完全一样的 call，可以利用 call#clone 方法进行克隆。

2）利用 client.dispatcher().enqueue(this) 来进行实际执行，dispatcher 是刚才看到的 OkHttpClient.Builder 的成员之一

3）AsyncCall是RealCall的一个内部类并且继承NamedRunnable，那么首先看NamedRunnable类是什么样的，如下：

```
public abstract class NamedRunnable implements Runnable {
  ......

  @Override 
  public final void run() {
   ......
    try {
      execute();
    }
    ......
  }

  protected abstract void execute();
}
```

而后在其execute（）方法中使用

```
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```

接下来观察getResponseWithInterceptorChain（），该方法就是使用拦截器对请求进行处理

```
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
```

其中CallServerInterceptor是最后一个Interceptor，用于进行网络请求，其他的是可以对头部进行一些处理


查看chain.proceed(request)方法

AsyncCall实现了execute方法，首先是调用getResponseWithInterceptorChain()方法获取响应，然后获取成功后，就调用回调的onReponse方法，如果失败，就调用回调的onFailure方法。最后，调用Dispatcher的finished方法。

关键代码：

responseCallback.onFailure(RealCall.this, new IOException(“Canceled”));

和

responseCallback.onResponse(RealCall.this, response);

走完这两句代码会进行回调到刚刚我们初始化Okhttp的地方
