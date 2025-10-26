---
创建时间: "2025-08-28 16:01:34"
作者: wangxiaoming
tags:
---
#### 一、核心架构设计
##### 1.分层架构
`OKHttp`采用清晰的分层设计，从应用层到网络层依次为：
- ​**应用层**​：业务代码通过`OkHttpClient`发起请求
- ​**拦截器链**​：处理请求/响应的模块化组件（责任链模式）
- ​**连接池**​：管理`TCP`连接复用
- ​**传输层**​：支持HTTP/1.1、HTTP/2、`WebSocket`等协议
- ​**Socket层**​：底层网络通信实现
##### 2.责任链模式（Interceptor  Chain）
请求和响应通过多个拦截器依次处理，每个拦截器专注于单一职责（如重试、缓存、协议转换等）。例如：
- `RetryAndFollowUpInterceptor`：处理重试和重定向
- `CacheInterceptor`：管理缓存策略
- `CallServerInterceptor`：执行实际网络I/O

#### 二、关键设计模式
##### 1.建造者模式（Builder）
用于构建复杂对象（如`Request`和`OkHttpClient`），支持链式调用简化配置。
```java
Request request = new Request.Builder()
    .url("https://api.example.com")
    .build();
```
##### 2.工厂模式（Factory）
通过`Call.Factory`接口创建`RealCall`对象，解耦对象创建逻辑。
##### 3.门面模式（Facade）
`OkHttpClient`作为统一入口，封装连接池、分发器等内部组件，简化调用。

#### 三、核心功能模块
##### 1.连接池管理
- ​**复用机制**​：通过`LRU`算法管理空闲连接，减少`TCP`握手开销
- ​**多路复用**​：HTTP/2支持单连接并发处理多个请求
- **连接有效性检测**​：定期清理无效连接，保证资源健康
##### 2.拦截器链
- ​**执行顺序**​：请求从前往后处理，响应从后往前返回
- ​**功能扩展**​：开发者可自定义拦截器实现日志记录、认证等
##### 3.异步调度机制
-  **Dispatcher**​：管理异步请求的线程池，控制并发数（默认最大64）
- ​**优先级调度**​：支持按请求优先级分配线程资源

#### 四、性能优化设计
##### 1.零拷贝技术
通过`Okio`库直接操作内存缓冲区，减少数据复制开销。
##### 2.HTTP/2支持
- **头部压缩**​：使用`HPACK`算法减少传输数据量
- ​**流优先级**​：动态调整请求处理顺序
##### 3.缓存策略
- ​**内存/磁盘缓存**​：根据`Cache-Control`策略自动管理缓存
- ​**透明`GZIP`压缩**​：自动解压响应数据
#### 五、异常处理与恢复
##### 1.重试机制
- 自动重试网络超时、服务器错误等可恢复异常
- 最多重试20次（默认5次）
##### 2.重定向处理
支持HTTP `3xx`状态码自动跳转，最多10次重定向循环检测。

#### 六、扩展性设计
##### 1.自定义拦截器
开发者可插入自定义逻辑（如鉴权、埋点）：
```java
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(chain -> {
        Request request = chain.request().newBuilder()
            .header("Authorization", "Bearer token")
            .build();
        return chain.proceed(request);
    })
    .build();
```
##### 2.协议扩展
支持通过`Protocol`接口扩展自定义协议（如`QUIC`）。