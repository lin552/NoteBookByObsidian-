---
创建时间: 2025-04-24 17:25:33
作者: wangxiaoming
tags:
  - Picasso
  - 图片框架
---
Picasso 是 Square 公司推出的轻量级图片加载库，以**简单易用、功能全面**著称，广泛用于 Android 应用中处理图片加载、缓存、变换等场景。其核心考点围绕**基础使用**、**缓存机制**、**性能优化**及**与其他库的对比**展开，以下是系统梳理：

#### 一、核心机制与基础使用
##### 1.核心组件​
- ​**Picasso 实例**​：全局单例，管理图片加载的全局配置（如缓存、线程池）。
- ​**Target**​：图片加载的目标（如 `ImageView`），负责接收加载结果（成功/失败）。
- ​**Request**​：图片加载的请求对象，封装 URL、变换参数、回调等。
##### ​2. 基础 API 使用​
```java
// 1. 初始化 Picasso（全局单例）
Picasso picasso = Picasso.get();

// 2. 加载网络图片到 ImageView
picasso.load("https://example.com/image.jpg")
    .placeholder(R.drawable.placeholder)  // 占位图（加载中）
    .error(R.drawable.error)              // 错误图（加载失败）
    .into(imageView);                     // 目标 ImageView

// 3. 加载本地资源图片
picasso.load(R.drawable.local_image)
    .resize(200, 200)                     // 调整尺寸
    .centerCrop()                         // 居中裁剪
    .into(imageView);
```

#### 二、缓存机制与内存管理
##### ​1. 缓存策略​

Picasso 采用**两级缓存**​（内存缓存 + 磁盘缓存），遵循 HTTP 缓存规范（RFC 7234），支持强缓存和协商缓存。
- ​**内存缓存**​：
    - 存储解码后的 Bitmap，使用 LRU（最近最少使用）算法淘汰旧数据。
    - 默认大小为可用内存的 15%（可通过 `MemoryCache` 自定义）。
- ​**磁盘缓存**​：
    - 存储压缩后的图片文件（如 `JPEG/PNG`），默认启用（需设备支持）。
    - 默认大小为 `50MB`（可通过 `DiskCache` 自定义路径和大小）。

##### ​**2. 内存优化**​
- ​**自动回收**​：当内存不足时，Picasso 会自动回收未被使用的 Bitmap，减少 `GC` 压力。
- ​**避免重复加载**​：同一 URL 的多次请求会被合并，仅加载一次。

#### 三、高级特性与实战技巧
##### ​1. 图片变换（Transformations）​​
Picasso 支持丰富的图片变换操作，通过 `Transformation` 接口实现（可自定义）：
- ​**内置变换**​：
    ```java
    // 圆形裁剪
    .transform(new CircleTransform())
    // 圆角裁剪（半径 10dp）
    .transform(new RoundedCornersTransform(10))
    // 模糊（半径 25）
    .transform(new BlurTransform(context, 25))
    ```

- ​**自定义变换**​：  
    实现 `Transformation` 接口的 `transform()` 和 `key()` 方法，例如添加水印：
    ```java
    public class WatermarkTransform implements Transformation {
        @Override
        public Bitmap transform(Bitmap source) {
            // 添加水印逻辑
            return watermarkedBitmap;
        }
    
        @Override
        public String key() {
            return "watermark"; // 唯一标识变换类型
        }
    }
    ```

##### ​2. 渐进式加载（Progressive Loading）​​
支持 JPEG 的渐进式加载（分阶段显示模糊到清晰的图片），提升用户体验：
```java
// 启用渐进式加载（需服务器返回渐进式 JPEG）
picasso.load(url)
    .progressive()
    .into(imageView);
```
##### ​3. 与 `RecyclerView` 集成​
在列表（如 `RecyclerView`）中加载图片时，需注意**复用问题**​（同一 `ImageView` 可能被重复绑定不同 URL）。Picasso 自动处理此问题，但需确保：
- 在 `onBindViewHolder` 中调用 `Picasso.get().load(url).into(viewHolder.imageView)`。
- 在 `onViewRecycled` 中取消未完成的请求（可选）：
    ```java
    @Override
    public void onViewRecycled(ViewHolder holder) {
        Picasso.get().cancelRequest(holder.imageView);
        super.onViewRecycled(holder);
    }
    ```

#### 四、高频面试题
1. ​**Picasso 如何避免 `OOM`（内存溢出）？​**​
    - 通过 `LRU` 内存缓存限制 Bitmap 数量。
    - 自动回收未被使用的 Bitmap，减少内存占用。
2. ​**Picasso 的缓存策略是什么？如何配置？​**​
    - 两级缓存（内存 + 磁盘），内存缓存默认 15% 可用内存，磁盘缓存默认 `50MB`。可通过 `Picasso.Builder` 自定义：
        ```java
        Picasso picasso = new Picasso.Builder(context)
            .memoryCache(new LruCache(20 * 1024 * 1024)) // 20MB 内存缓存
            .build();
        ```
3. ​**如何实现图片的圆形裁剪？​**​
    - 使用 `CircleTransform`（内置）或自定义 `Transformation`。
4. ​**Picasso 与 Glide 的区别？​**​
    - ​**轻量性**​：Picasso 更轻量（约 `100KB`），Glide 更重（约 `200KB`）。
    - ​**功能**​：Glide 支持 GIF、视频帧，Picasso 仅支持静态图。
    - ​**缓存**​：Glide 缓存策略更灵活（如内存/磁盘缓存分离），Picasso 依赖 `LRU`。
5. ​**如何取消 Picasso 的未完成请求？​**​
    - 调用 `Picasso.get().cancelRequest(Target target)` 或 `cancelRequest(Request request)`。

#### 五、源码解析与最佳实践
##### ​1. 请求流程​
Picasso 的请求流程可概括为：
1. 构建 `Request` 对象（URL、变换参数等）。
2. 通过 `Picasso` 实例提交请求，生成 `Target`。
3. 检查内存缓存（命中则直接显示）。
4. 未命中则从磁盘缓存加载（命中则解码后显示）。
5. 磁盘缓存未命中则发起网络请求，下载后解码并缓存。
##### ​2. 最佳实践​
- ​**避免主线程阻塞**​：Picasso 自动在后台线程处理网络请求和解码，无需手动切换线程。
- ​**合理设置图片尺寸**​：通过 `resize(width, height)` 调整图片大小，避免加载过大的 Bitmap。
- ​**使用占位图**​：提升用户体验，避免空白区域闪烁。