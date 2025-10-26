---
创建时间: 2025-04-24 11:22:06
作者: wangxiaoming
tags:
  - Glide
  - 图片框架
---
#### 一、核心即使与基础使用
1. **缓存机制**​
    - ​**内存缓存**​：
        - ​`LruResourceCache`**​：基于`LRU`算法管理最近使用的图片资源，默认占可用内存的40%
        - ​**弱引用缓存（`ActiveResources`）​**​：存储当前显示的图片，防止被`GC`回收
    - ​**磁盘缓存**​：
        - ​`DiskLruCache`**​：存储压缩后的图片，默认策略为`DiskCacheStrategy.ALL`（缓存原始图和转换后的图）
        - ​**自定义缓存策略**​：通过`diskCacheStrategy()`设置（如`NONE`、`SOURCE`、`RESULT`）
2. ​**加载与显示**​
    - ​**基础API**​：
        ```java
        Glide.with(context)
             .load(url)
             .placeholder(R.drawable.placeholder)  // 占位图
             .error(R.drawable.error)              // 错误图
             .into(imageView);
        ```
    - ​**图片变换**​：
        - 圆形裁剪：`transform(CircleCrop())`
        - 圆角处理：`transform(RoundedCorners(10))`
    - ​**缩略图**​：
        ```java
        .thumbnail(0.1f)  // 加载原图的10%尺寸作为缩略图
        ```

#### 二、性能优化与高级特性
1. ​**内存优化**​
    - ​**Bitmap复用**​：通过`BitmapPool`复用内存块，减少`GC`频率
    - ​**动态压缩**​：根据`ImageView`尺寸自动缩放图片，避免内存浪费
    - ​**大图处理**​：
        - 限制解码尺寸：`.override(width, height)`
        - 使用低内存格式：`.format(DecodeFormat.PREFER_RGB_565)`
2. ​**磁盘缓存优化**​
    - ​**冷热数据分离**​：按业务场景划分缓存池（如头像与高清图分开存储）
    - ​**加密保护**​：对敏感缩略图进行`AES-256`加密
3. ​**并发与生命周期管理**​
    - ​**优先级调度**​：根据滑动速度动态调整加载优先级（高速滑动时加载缩略图）
    - ​**生命周期集成**​：自动取消未完成的请求，避免内存泄漏

#### 三、高频面试题
1. ​**Glide如何避免`OOM`？​**​
    - 动态压缩图片尺寸，使用`RGB_565`格式减少内存占用。
    - 通过`BitmapPool`复用Bitmap内存块
2. ​**Glide的缓存策略有哪些？​**​
    - `DiskCacheStrategy.NONE`（不缓存）、`SOURCE`（仅原始图）、`RESULT`（仅转换后图）、`ALL`（全部缓存）
3. ​**如何监控Glide性能？​**​
    - 使用`RequestListener`监听加载事件。
    - 通过`Android Profiler`分析内存与CPU占用
4. ​**Glide与Picasso的区别？​**​
    - Glide支持GIF和视频截图，Picasso仅支持静态图。
    - Glide默认使用`LRU`内存缓存，Picasso依赖`LruCache`手动配置
```java
@GlideModule
public class MyAppGlideModule extends AppGlideModule {
    @Override
    public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
        builder.setMemoryCache(new LruResourceCache(20 * 1024 * 1024)); // 20MB内存缓存
    }
}
```
#### 四、源码解析与实战技巧
1. ​**内存缓存实现**​
    - ​**双重缓存结构**​：
        - `LruResourceCache`：强引用缓存，存储转换后的图片。
        - `ActiveResources`：弱引用缓存，存储当前显示的图片
2. ​**磁盘缓存流程**​
    - 生成唯一缓存键（`EngineKey`），结合URL、尺寸、转换参数生成哈希值。
    - 写入时先存原始数据，再存转换后数据（若策略为`ALL`）
3. ​**实战优化方案**​
    - ​**预加载策略**​：
        ```java
        Glide.with(context).load(url).preload();  // 预加载到缓存
        ```
    - ​**智能降级**​：根据网络状态动态切换图片质量
