---
创建时间: 2025-04-15 22:35:30
作者: wangxiaoming
tags:
  - Retrofit
  - Android
  - 网络
---
#### 一、核心机制与设计思想
1. ​**动态代理与接口映射**​
    - ​**动态代理**​：Retrofit 通过 `JDK`动态代理（`Proxy.newProxyInstance`）生成接口的代理类，拦截方法调用并转换为 HTTP 请求。核心类 `ServiceMethod` 负责解析接口方法的注解，生成对应的 `Call` 对象
    - ​**适配器模式**​：通过 `CallAdapter.Factory` 将原始 `Call` 转换为其他形式（如 `RxJava` 的 `Observable` 或 Kotlin 协程的 `Deferred`）
    
2. ​**转换器（Converter）​**​
    - ​**作用**​：将请求参数序列化为 `JSON/XML` 等格式，或将响应体反序列化为 Java 对象。
    - ​**常用实现**​：`GsonConverterFactory`（`JSON`）、`MoshiConverterFactory`（`Moshi`）、`ScalarsConverterFactory`（字符串）

3. ​`OkHttp` 集成**​
    - 底层默认使用 `OkHttp` 作为网络传输层，支持连接池、缓存、拦截器等特性。
    - 可通过 `callFactory` 替换为自定义的 HTTP 客户端（如 `OkHttp` 或自定义实现）

#### 二、核心组件与使用
1. ​**Retrofit 实例构建**​
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.example.com/")  // 基础 URL
    .addConverterFactory(GsonConverterFactory.create())  // 转换器
    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())  // 适配器
    .client(okHttpClient)  // 自定义 OkHttp 客户端
    .build();
```
1. ​**接口定义与注解**​
    - ​**请求方法注解**​：`@GET`、`@POST`、`@PUT`、`@PATCH`、`@DELETE`。
    - ​**参数注解**​：
        - `@Path`：动态替换 URL 路径参数（如 `/users/{id}` → `id="123"`）。
        - `@Query`：添加查询参数（如 `?name=John`）。
        - `@Body`：发送 `JSON` 等数据体（需配合转换器）。
        - `@Header`：添加请求头。
    - ​**标记注解**​：`@FormUrlEncoded`（表单提交）、`@Multipart`（文件上传）
2. ​**请求执行**​
    - ​**同步请求**​：`call.execute()`（阻塞当前线程）。
    - ​**异步请求**​：`call.enqueue(Callback)`（非阻塞，回调在后台线程）。

#### 三、高频考点与原理剖析
1. ​**动态代理原理**​
    - 通过 `Retrofit.create(Class<T> service)` 生成代理类，拦截接口方法调用，解析注解生成 `ServiceMethod`，最终调用 `OkHttp` 发送请求
2. ​**线程切换与调度**​
    - 默认在后台线程执行网络请求，结果回调到主线程（需配置 `callbackExecutor`）。
    - 结合 `RxJava` 或 Kotlin 协程实现更灵活的线程调度。
3. ​**缓存机制**​
    - 依赖 `OkHttp` 的缓存策略，通过 `Cache` 类配置磁盘缓存目录和大小。
    - 支持强缓存（`Cache-Control: max-age`）和协商缓存（`ETag`/`Last-Modified`）
4. ​**多 Base URL 处理**​
    - 方案一：通过注解 `@Url` 动态指定完整 URL。
    - 方案二：为不同域名创建多个 Retrofit 实例。
    - 方案三：自定义 `CallAdapter` 动态切换 Base URL（如`JessYan` 方案）
5. ​**拦截器与日志**​
    - 添加 `HttpLoggingInterceptor` 打印请求/响应日志。
    - 自定义拦截器实现鉴权、重试、数据加密等
#### 四、设计模式与源码解析
1. ​**设计模式应用**​
    - ​**动态代理**​：接口方法调用转换为 HTTP 请求。
    - ​**建造者模式**​：通过 `Retrofit.Builder` 逐步配置参数。
    - ​**适配器模式**​：将 `Call` 转换为不同响应类型（如 `RxJava`、协程）。
    - ​**策略模式**​：根据注解选择不同的请求策略（如 GET/POST）

2. ​**关键类解析**​
    - `Retrofit`：核心配置类，持有 `ServiceMethodCache`、`CallFactory` 等。
    - `ServiceMethod`：解析接口方法注解，生成请求参数和 `Call`。
    - `CallAdapter`：处理响应类型转换（如 `Observable`、`LiveData`）
#### 五、性能优化与实战技巧
1. ​**连接池优化**​
    - 配置 `OkHttp` 的 `ConnectionPool`，复用 TCP 连接，减少握手开销。
    ```java
    ConnectionPool connectionPool = new ConnectionPool(5, 5, TimeUnit.MINUTES);
    OkHttpClient client = new OkHttpClient.Builder()
        .connectionPool(connectionPool)
        .build();
    ```
2. ​**内存泄漏预防**​
    - 使用 `Application` Context 替代 Activity Context。
    - 在 Activity/Fragment 销毁时取消未完成的请求（`call.cancel()`）。
3. ​**大文件下载**​
    - 启用流式响应：`@Streaming` 注解避免内存溢出。
    - 分块下载：通过 `Range` 头实现断点续传。
4. ​**多线程并发控制**​
    - 配置 `OkHttp` 的 `Dispatcher`，限制最大并发请求数。
    ```java
    Dispatcher dispatcher = new Dispatcher();
    dispatcher.setMaxRequests(100);  // 最大并发数
    dispatcher.setMaxRequestsPerHost(10);  // 单主机最大并发数
    ```
#### 六、高频面试题
1. ​**Retrofit 的动态代理是如何实现的？​**​
    - 通过 `Proxy.newProxyInstance` 生成接口代理类，拦截方法调用并解析注解生成请求
2. ​**Retrofit 如何支持多线程并发？​**​
    - 依赖 OkHttp 的 `Dispatcher` 线程池管理请求，通过配置最大并发数和主机并发数优化性能。
3. ​**如何实现 Retrofit 的全局错误处理？​**​
    - 自定义 `Interceptor` 拦截所有响应，统一处理错误码和异常。
4. ​**Retrofit 与 `OkHttp` 的关系？​**​
    - Retrofit 基于 `OkHttp` 实现网络请求，`OkHttp` 负责底层 `TCP` 连接、缓存、重试等。
5. ​**如何优化 Retrofit 的性能？​**​
    - 启用连接池复用、压缩请求体、使用 `RGB_565` 格式解析图片、配置合理的线程池。