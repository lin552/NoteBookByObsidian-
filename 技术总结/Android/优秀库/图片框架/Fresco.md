---
创建时间: 2025-04-24 17:11:32
作者: wangxiaoming
tags:
  - Fresco
  - 图片框架
---
#### 一、核心机制与基础使用
1. ​**组件架构**​
    - ​`Drawee`​：负责图片显示与状态管理（如占位图、错误图），通过 `SimpleDraweeView` 实现。
    - ​`ImagePipeline`​：管理图片加载流程，包括网络请求、解码、缓存等核心逻辑。
    - ​`Cache`​：三级缓存体系（内存缓存、编码缓存、磁盘缓存），优化加载效率
2. ​**基础API**​
    ```java
    // 加载网络图片
    Fresco.initialize(context);
    SimpleDraweeView draweeView = findViewById(R.id.my_image_view);
    draweeView.setImageURI("https://example.com/image.jpg");
    ```
    - ​**占位图与错误图**​：通过 `setPlaceholderImage()` 和 `setErrorImage()` 设置。
#### 二、内存管理与优化
1. ​`Ashmem` 匿名共享内存**​
    - ​**原理**​：将 Bitmap 存储在 Android 系统管理的 `Ashmem` 区域，避免 Java 堆内存溢出（`OOM`）
    - ​**优势**​：减少 `GC` 压力，支持低内存设备流畅运行。
2. ​**内存缓存策略**​
    - ​`LruMemoryCache`​：基于 `LRU` 算法管理解码后的 Bitmap，通过 `MemoryCacheParams` 配置最大容量和淘汰策略
    - ​**编码缓存**​：存储未解码的原始数据（如字节流），避免重复下载
3. ​**内存优化技巧**​
    - ​**按需缩放**​：通过 `ResizeOptions` 调整图片尺寸，减少内存占用
    - ​**复用 Bitmap**​：利用 `CloseableReference` 管理生命周期，防止内存泄漏。
#### 三、缓存机制与配置
1. ​**三级缓存结构**

| ​**缓存层**​  | ​**存储内容**​          | ​**特点**​        |
| ---------- | ------------------- | --------------- |
| ​**内存缓存**​ | Bitmap、EncodedImage | 快速访问，LRU 淘汰策略   |
| ​**编码缓存**​ | 原始图片数据（字节流）         | 避免重复解码          |
| ​**磁盘缓存**​ | 压缩后的图片文件            | 持久化存储，支持小图/大图分离 |

2. ​**磁盘缓存配置**​
    ```java
    DiskCacheConfig diskCacheConfig = DiskCacheConfig.newBuilder(context)
        .setMaxCacheSize(100 * 1024 * 1024) // 100MB
        .build();
    ImagePipelineConfig config = ImagePipelineConfig.newBuilder(context)
        .setMainDiskCacheConfig(diskCacheConfig)
        .build();
    Fresco.initialize(context, config);
    ```

#### 四、图片加载流程解析
1. ​**Producer 链式处理**​
    - ​**流程**​：`NetworkFetcher` → `DecodeProducer` → `ResizeProducer` → `PostprocessorProducer`。
    - ​**异步解码**​：在后台线程完成图片解码，避免阻塞 `UI`
2. ​**渐进式加载**​
    - ​**支持格式**​：JPEG、`WebP`。
    - ​**实现**​：`ProgressiveJpegDecoder` 分阶段解析数据，逐步显示图片

#### 五、动态图与高级特性
1. ​`GIF/WebP` 动画支持**​
    - ​`AnimatedDrawable`**​：通过 `AnimatedDrawable2` 实现帧动画控制。
    - ​**内存优化**​：自动回收无效帧，减少资源占用
2. ​**自定义扩展**​
    - ​**自定义 Producer**​：扩展网络请求或数据处理逻辑。
    - ​**自定义 `Postprocessor`​：添加滤镜、裁剪等后处理效果

#### 六、性能优化与实战技巧
1. **避免内存泄漏**​
    - ​**及时释放资源**​：在 `onDestroy()` 中调用 `controller.unbindDraweeHierarchy()`。
    - ​**弱引用管理**​：使用 `WeakReference` 弱引用 View，防止持有导致泄漏。
2. ​**网络优化**​
    - ​**HTTP/2 支持**​：启用多路复用，减少请求延迟。
    - ​**重试策略**​：配置网络请求重试次数与超时时间。
3. ​**对比 Glide**​
    - ​**优势**​：Fresco 内存管理更优（`Ashmem`）、支持渐进式加载。
    - ​**劣势**​：体积较大，配置复杂度高。
#### 七、高频面试题
1. ​**Fresco 如何解决 `OOM` 问题？​**​
    - 通过 `Ashmem` 存储 Bitmap，结合 `LRU` 缓存策略控制内存占用
2. ​**Fresco 的三级缓存如何工作？​**​
    - 优先从内存缓存读取，未命中则查询编码缓存，最后访问磁盘缓存
3. ​**如何实现图片的渐进式加载？​**​
    - 使用 `ProgressiveJpegDecoder` 分阶段解码，逐步渲染到`UI`
4. ​**Fresco 与 Glide 的区别？​**​
    - Fresco 内存管理更高效，支持动态图；Glide 更轻量，集成更简单。