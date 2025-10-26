---
创建时间: "2025-08-25 14:02:47"
作者: wangxiaoming
tags:
---
#### 一、网络传输优化
##### 1.协议与`CDN`选择
- ​**启用 HTTP/2 或 HTTP/3**​：通过 `HttpEngine`或 `Cronet`网络栈支持多路复用和头部压缩，减少网络延迟
- ​`CDN` 加速**​：结合业务场景选择低延迟 `CDN` 节点，动态切换最优线路。
```kotlin
// 使用 Cronet 网络栈（需集成 Cronet 库）
val cronetDataSourceFactory = CronetDataSource.Factory(cronetEngine, executor)
val dataSourceFactory = DefaultDataSource.Factory(context, cronetDataSourceFactory)
```

##### 2.动态重试与超时设置
- ​**网络异常重试**​：通过 `RetryDataSource`实现失败后自动重试。
- **超时配置**​：调整连接和读取超时时间，避免长时间等待。
```kotlin
val dataSourceFactory = DefaultDataSource.Factory(context)
    .setDefaultRequestProperties(mapOf(
        "timeout" to "5000",  // 5秒超时
        "retry-count" to "3"  // 重试3次
    ))
```

#### 二、缓存与加载缓存优化
##### 1.自适应缓存参数
•**动态调整缓冲时间**​：根据网络状态调整 `BufferForPlaybackMs`和 `BufferForPlaybackAfterRebufferMs`。
```kotlin
val loadControl = DefaultLoadControl.Builder()
    .setBufferForPlaybackMs(2000)  // 播放前缓冲2秒
    .setBufferForPlaybackAfterRebufferMs(1000)  // 卡顿后缓冲1秒
    .build()
player = ExoPlayer.Builder(context)
    .setLoadControl(loadControl)
    .build()
```

##### 2.预加载与分片缓存
- ​**预加载关键数据**​：通过 `ProgressiveMediaSource`预加载视频头信息，加速启播。
- **分片缓存策略**​：使用 `CacheDataSource`缓存已加载片段，减少重复请求
```kotlin
val cache = SimpleCache(cacheDir, LeastRecentlyUsedCacheEvictor(100 * 1024 * 1024)) // 100MB 缓存
val cacheDataSourceFactory = CacheDataSource.Factory()
    .setCache(cache)
    .setUpstreamDataSourceFactory(dataSourceFactory)
```

#### 三、解码与渲染优化
##### 1.硬件解码与解码器复用
- ​**启用 `MediaCodec` 硬解**​：通过 `DefaultRenderersFactory`设置硬件解码。
- **解码器复用**​：避免频繁创建/销毁解码器实例，减少启播延迟
```kotlin
val renderersFactory = DefaultRenderersFactory(context)
    .setEnableDecoderFallback(true)  // 启用解码器回退机制
player = ExoPlayer.Builder(context, renderersFactory).build()
```
##### 2.零延迟渲染（直播场景）
•**禁用音视频同步**​：通过 `setAudioSyncParams`调整同步策略，降低直播延迟。
```kotlin
player.setAudioSyncParams(
    audioSyncParams = AudioSyncParams(
        toleranceMs = 20,  // 允许20ms偏差
        syncMode = AudioSyncMode.AUDIO_MASTER
    )
)
```

#### 四、内存与性能优化
##### 1.避免内存泄漏
•**生命周期管理**​：在 `Activity/Fragment`销毁时释放播放器资源。
```kotlin
override fun onDestroy() {
    player.release()
    super.onDestroy()
}
```

##### 2.降低CPU/GPU负载
- ​**优化视频编码格式**​：优先使用 `H.265/VP9` 等高压缩率格式。
- ​**动态降分辨率**​：根据设备性能切换分辨率（如`720P → 480P`）。
```kotlin
val trackSelector = DefaultTrackSelector(context).apply {
    setParameters(
        parameters.buildUponParameters()
            .setMaxVideoSize(1280, 720)  // 限制分辨率
    )
}
player = ExoPlayer.Builder(context)
    .setTrackSelector(trackSelector)
    .build()
```

#### 五、缓存与数据管理
##### 1.智能换成策略
- ​`LRU` 缓存淘汰**​：基于访问频率淘汰冷数据，保留高频内容。
- ​**磁盘缓存优化**​：使用 `SimpleCache`结合数据库管理缓存元数据
```kotlin
val databaseProvider = StandaloneDatabaseProvider(context)
val cache = SimpleCache(cacheDir, LeastRecentlyUsedCacheEvictor(50 * 1024 * 1024), databaseProvider)
```
##### 2.流式数据解析
•**提前解析元数据**​：通过 `MediaMetadataRetriever`预加载视频时长、码率等信息。
```kotlin
val retriever = MediaMetadataRetriever()
retriever.setDataSource(videoUrl)
val durationMs = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION)?.toLong() ?: 0
```

#### 六、错误处理与容错
##### 1.网络错误恢复
•**自动重试策略**​：结合 `RetryDataSource`实现指数退避重试。
```kotlin
val retryDataSourceFactory = RetryDataSource.Factory(dataSourceFactory)
    .setMaxRetries(3)
    .setRetryDelayMs(1000)
```

##### 2.异常帧丢弃
•**跳过损坏数据**​：通过 `Parser`拦截错误帧并继续播放。
```kotlin
val mediaSource = ProgressiveMediaSource.Factory(dataSourceFactory)
    .createMediaSource(MediaItem.fromUri(videoUrl))
player.setMediaSource(mediaSource)
```

#### 七、高级功能优化
##### 1.动态码率切换（`ABR`）
•**自适应流媒体**​：使用 `HLS/DASH` 协议动态调整码率。
```kotlin
val mediaSource = DashMediaSource.Factory(dataSourceFactory)
    .createMediaSource(MediaItem.fromUri(hlsUrl))
```

##### 2.后台播放与锁屏控制
•​**后台服务播放**​：通过 `Service`保持播放状态。
```kotlin
class PlaybackService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        player.playWhenReady = true
        return START_STICKY
    }
}
```

#### 八、性能监控与调试
##### 1.实时性能指标
•**使用 `ExoPlayer` 的 `DebugLogger`**​：输出解码、渲染等关键日志。
```kotlin
ExoPlayer.Builder(context)
    .setVerboseLoggingEnabled(true)
    .build()
```

##### 2.`Android Profiler`分析
- ​**CPU/内存占用监控**​：定位解码、渲染等高耗时操作。
- ​**网络流量分析**​：通过 `Network Profiler`检查数据加载效率。