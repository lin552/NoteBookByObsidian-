---
创建时间: 2025-04-15 22:34:29
作者: wangxiaoming
tags:
  - OKHttp
  - Android
  - 网络
---
`OkHttp` 是 Square 公司推出的高性能 HTTP 客户端库，广泛用于 Android 和 Java 应用，以**高效、灵活、易扩展**著称。其核心设计目标是简化网络请求，同时提供强大的底层控制能力。以下是其核心考点的系统梳理：

#### 一、核心组件与基础使用
##### 1. 核心类与流程​
`OkHttp` 的核心组件包括 `OkHttpClient`、`Request`、`Call`、`Response`，请求流程可概括为：  
​**构建 `OkHttpClient` → 创建 `Request` → 发起 `Call` → 处理 `Response`**。
```java
// 1. 构建 OkHttpClient（全局单例）
OkHttpClient client = new OkHttpClient.Builder()
    .connectTimeout(10, TimeUnit.SECONDS) // 连接超时
    .readTimeout(30, TimeUnit.SECONDS)    // 读取超时
    .build();

// 2. 创建 Request（GET/POST）
Request request = new Request.Builder()
    .url("https://api.example.com/data")
    .get() // 默认 GET，可替换为 post()
    .build();

// 3. 发起请求（同步/异步）
Call call = client.newCall(request);
// 同步请求（阻塞当前线程）
Response response = call.execute();
// 异步请求（非阻塞，回调在后台线程）
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        // 失败处理（如网络错误）
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        // 成功处理（响应体需手动关闭）
        String result = response.body().string();
    }
});
```
##### 2. 请求与响应的构建​
- ​**`Request` 构建**​：支持 URL、方法（GET/POST/PUT/DELETE 等）、请求头、请求体（`RequestBody`）。
    - ​**POST 请求体**​：常用 `FormBody`（表单）、`MultipartBody`（文件上传）、`RequestBody.create()`（自定义）。
        ```java
        // 表单提交
        RequestBody formBody = new FormBody.Builder()
            .add("username", "user")
            .add("password", "pass")
            .build();
        Request request = new Request.Builder()
            .url("https://api.example.com/login")
            .post(formBody)
            .build();
        
        // 文件上传（Multipart）
        RequestBody fileBody = RequestBody.create(MediaType.get("image/jpeg"), new File("photo.jpg"));
        MultipartBody multipartBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("file", "photo.jpg", fileBody)
            .build();
        ```
- ​**`Response` 处理**​：通过 `response.code()` 获取状态码，`response.headers()` 获取响应头，`response.body().string()` 获取响应体（仅能调用一次）。

#### 二、核心机制与高级特性
##### 1. 连接池（Connection Pool）​​
`OkHttp` 通过**连接池**复用`TCP` 连接，减少握手开销，提升性能。核心参数：
- `maxIdleConnections`：最大空闲连接数（默认 5）。
- `keepAliveDuration`：空闲连接的存活时间（默认 5 分钟）。
​**作用**​：
- 复用已建立的 `TCP` 连接，避免重复三次握手。
- 限制空闲连接数，防止资源浪费。
```java
ConnectionPool connectionPool = new ConnectionPool(
    10, // maxIdleConnections：最大空闲连接数
    30, // keepAliveDuration：空闲连接存活时间（秒）
    TimeUnit.SECONDS
);

OkHttpClient client = new OkHttpClient.Builder()
    .connectionPool(connectionPool)
    .build();
```
##### ​2. 缓存机制（Cache）​​
`OkHttp` 支持**内存缓存**和**磁盘缓存**，通过 `Cache` 类配置，遵循 HTTP 缓存规范（RFC 7234）。
- ​**内存缓存**​：存储小文件（如图片），默认无限制（需手动设置）。
- ​**磁盘缓存**​：存储大文件（如 `APK`、图片），需指定缓存目录和大小（默认 0，需手动启用）。
​**配置示例**​：
```java
// 磁盘缓存（需创建缓存目录）
File cacheDir = new File(context.getCacheDir(), "http-cache");
int cacheSize = 10 * 1024 * 1024; // 10MB
Cache cache = new Cache(cacheDir, cacheSize);

OkHttpClient client = new OkHttpClient.Builder()
    .cache(cache) // 启用磁盘缓存
    .build();
```
​**缓存策略**​：
- 强缓存（`Cache-Control: max-age=3600`）：直接使用缓存，不请求服务器。
- 协商缓存（`Cache-Control: no-cache`）：请求服务器验证缓存是否有效（通过 `ETag` 或 `Last-Modified`）。
##### ​3. 拦截器（Interceptor）​​
拦截器是 `OkHttp` 最灵活的扩展机制，可在请求发送前或响应接收后修改请求/响应。分为**应用层拦截器**和**网络层拦截器**​：
- ​**应用层拦截器**​：在请求发送前修改（如添加公共头、日志记录），不感知底层网络细节（如重试、连接池）。
- ​**网络层拦截器**​：在请求经过连接池、重定向后修改（如处理 `HTTPS` 证书、自定义协议）。

​**示例：添加公共请求头**​
```java
Interceptor headerInterceptor = chain -> {
    Request originalRequest = chain.request();
    Request newRequest = originalRequest.newBuilder()
        .addHeader("Authorization", "Bearer token")
        .build();
    return chain.proceed(newRequest);
};

OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(headerInterceptor) // 应用层拦截器
    .build();

//网络层拦截器  
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            // 获取最终执行的请求（可能已被重定向、重试）
            Request finalRequest = chain.request();

            // 记录请求 URL（可能是重定向后的 URL）
            Log.d("NetworkInterceptor", "最终请求 URL：" + finalRequest.url());

            // 获取底层连接（如 HTTP/2 连接、HTTPS 连接）
            Connection connection = chain.connection();
            if (connection != null) {
                Log.d("NetworkInterceptor", "连接协议：" + connection.protocol()); // 输出：h2 或 http/1.1
            }

            // 继续执行请求链
            return chain.proceed(finalRequest);
        }
    })
    .build();
```

​**典型用途**​：
- 日志记录（`HttpLoggingInterceptor`）。
- 动态修改请求参数（如添加时间戳防重放）。
- 统一处理错误（如 401 未授权时跳转登录）。
##### ​4. 异步请求与线程管理​
`OkHttp` 的异步请求通过 `enqueue()` 方法实现，回调在 ​**Dispatcher 线程池**中执行（默认核心线程数 0，最大线程数 256）。
- ​**Dispatcher 作用**​：管理异步请求的线程，避免阻塞主线程。
- ​**取消请求**​：通过 `Call.cancel()` 取消未完成的请求（如 Activity 销毁时）。

##### 5.线程池实现
```java
public class Dispatcher {
    // 任务队列（待执行的任务）
    private final Deque<RealCall.AsyncCall> readyAsyncCalls = new ArrayDeque<>();
    // 正在执行的任务
    private final Deque<RealCall.AsyncCall> runningAsyncCalls = new ArrayDeque<>();

    // 线程池（核心逻辑）
    private final ThreadPoolExecutor executorService;

    public Dispatcher() {
        // 初始化线程池（核心线程数 0，最大线程数 Integer.MAX_VALUE，空闲线程存活 60s，任务队列由 readyAsyncCalls 控制）
        executorService = new ThreadPoolExecutor(
            0,
            Integer.MAX_VALUE,
            60L,
            TimeUnit.SECONDS,
            new SynchronousQueue<>(), // 无界队列（实际由 readyAsyncCalls 控制并发）
            new ThreadFactory() {
                @Override
                public Thread newThread(Runnable r) {
                    Thread thread = new Thread(r, "OkHttp Dispatcher");
                    thread.setDaemon(true); // 守护线程（JVM 退出时自动终止）
                    return thread;
                }
            }
        );
    }

    // 提交任务（关键逻辑）
    void enqueue(AsyncCall call) {
        synchronized (this) {
            // 检查是否超过全局最大并发数
            if (runningAsyncCalls.size() >= maxRequests) {
                readyAsyncCalls.addLast(call); // 入队等待
                return;
            }
            // 检查是否超过单主机最大并发数
            String host = call.request().url().host();
            int hostCount = 0;
            for (AsyncCall c : runningAsyncCalls) {
                if (c.request().url().host().equals(host)) {
                    hostCount++;
                }
            }
            if (hostCount >= maxRequestsPerHost) {
                readyAsyncCalls.addLast(call); // 入队等待
                return;
            }
            // 未超限制，提交到线程池执行
            runningAsyncCalls.addLast(call);
        }
        executorService.execute(call); // 线程池执行任务
    }
}
```
自定义线程池配置
```java
// Java 示例
Dispatcher dispatcher = new Dispatcher();
dispatcher.setMaxRequests(20);       // 全局最大并发数
dispatcher.setMaxRequestsPerHost(3); // 单主机最大并发数

OkHttpClient client = new OkHttpClient.Builder()
    .dispatcher(dispatcher)
    .build();

// 发起 25 个异步请求（测试并发限制）
for (int i = 0; i < 25; i++) {
    Request request = new Request.Builder()
        .url("https://api.example.com/data") // 同一主机
        .build();
    client.newCall(request).enqueue(new Callback() {
        // ... 回调逻辑
    });
}
```
#### 三、性能优化与最佳实践
##### 1. 连接池调优​
- ​**减少空闲连接**​：根据业务场景调整 `maxIdleConnections`（如高并发场景设为 10，低并发设为 5）。
- ​**缩短存活时间**​：`keepAliveDuration` 设为 3-5 分钟（避免长时间占用连接）。

##### ​2. 缓存策略优化​
- ​**静态资源缓存**​：图片、`JS`、CSS 等静态资源设置长缓存（如 `max-age=31536000`）。
- ​**动态资源不缓存**​：接口数据设置 `no-cache` 或短缓存（如 `max-age=60`）。

##### ​3. 异步请求管理​
- ​**避免内存泄漏**​：在 Activity/Fragment 销毁时取消未完成的请求（通过 `call.cancel()`）。
- ​**批量请求合并**​：使用 `Call.enqueue()` 时，合并多个请求的回调（如使用 `CompositeDisposable`）。

##### ​4. `HTTPS` 安全配置​
- ​**证书校验**​：默认严格校验证书，可通过 `CertificatePinner` 自定义信任证书（如自签名证书）。
- `​TLS` 版本​：强制使用 `TLS 1.2+`（避免旧版本漏洞）。

#### 四、高频面试题
1. `​OkHttp` 的核心组件有哪些？各自的作用是什么？​**​  
    核心组件包括 `OkHttpClient`（客户端配置）、`Request`（请求构建）、`Call`（请求执行）、`Response`（响应处理）。
    
2. ​`OkHttp` 如何实现连接复用？​**​  
    通过连接池（`ConnectionPool`）管理 `TCP` 连接，复用空闲连接，减少三次握手开销。
    
3. ​`OkHttp` 的缓存机制是如何工作的？​**​  
    遵循 HTTP 缓存规范，支持强缓存和协商缓存。磁盘缓存存储响应体，内存缓存存储小文件。
    
4. ​**拦截器的作用是什么？应用层和网络层拦截器的区别？​**​  
    拦截器用于修改请求/响应。应用层拦截器在请求发送前修改（不感知底层网络），网络层拦截器在连接建立后修改（如处理重定向）。
    
5. ​**如何避免 `OkHttp` 异步请求的内存泄漏？​**​  
    在 Activity/Fragment 销毁时调用 `call.cancel()` 取消未完成的请求。
    
6. ​`OkHttp` 与 Retrofit 的关系？​**​  
    Retrofit 是基于 `OkHttp` 的 `RESTful API` 封装库，底层使用 `OkHttp` 处理网络请求，提供更简洁的注解式 API。


#### 五、源码解析与实战技巧
##### 1. 请求执行流程​
`OkHttp` 的请求执行核心是 `RealCall` 类，流程如下：
1. `RealCall` 通过 `Dispatcher` 提交到线程池。
2. 检查连接池是否有可用连接（复用或新建）。
3. 发送请求并等待响应。
4. 响应返回后，通过拦截器链处理（应用层 → 网络层）。

##### ​2. 自定义拦截器示例
```java
// 日志拦截器（打印请求/响应信息）
Interceptor loggingInterceptor = chain -> {
    Request request = chain.request();
    long startTime = System.currentTimeMillis();
    Response response = chain.proceed(request);
    long duration = System.currentTimeMillis() - startTime;

    Log.d("OkHttp", "Request: " + request.url());
    Log.d("OkHttp", "Response Code: " + response.code());
    Log.d("OkHttp", "Duration: " + duration + "ms");
    return response;
};
```