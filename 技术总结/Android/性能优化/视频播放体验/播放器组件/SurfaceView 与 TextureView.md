---
创建时间: 2025-07-30 12:01:38
作者: wangxiaoming
tags:
  - SurfaceView
  - TextureView
---
#### 一、`SurfaceView` 与 `TextureView` 的核心区别

| **特性**​      | ​**SurfaceView**​                                      | ​**TextureView**​                                  |
| ------------ | ------------------------------------------------------ | -------------------------------------------------- |
| ​**渲染机制**​   | 独立窗口渲染，通过双缓冲机制在独立线程绘制，不阻塞 UI 线程<br>                    | 集成在 View 层级中，依赖硬件加速，通过 `SurfaceTexture` 监听帧数据更新    |
| ​**布局灵活性**​  | 无法进行平移、缩放、旋转等变换，层级固定<br>                               | 支持 View 的所有属性（如动画、旋转、缩放），可动态调整位置和大小                |
| ​**生命周期管理**​ | 需手动处理 `SurfaceHolder` 的创建/销毁回调（如 `surfaceCreated`）<br> | 自动与 View 生命周期绑定，通过 `SurfaceTextureListener` 监听状态变化 |
| ​**性能**​     | 双缓冲机制减少卡顿，适合高帧率视频；内存占用较低<br>                           | 依赖硬件加速，渲染效率高，但频繁调整布局可能引发重绘开销                       |
| ​**适用场景**​   | 全屏播放、固定位置的视频展示（如直播流）<br>                               | 需要动态布局（如弹幕叠加、列表播放）、动画效果或与其他 View 交互的场景             |

#### 二、`IjkPlayer`中`SurfaceView`的应用
##### 1.实现原理
- ​**独立渲染层**​：通过 `SurfaceHolder` 获取 Surface，将解码后的视频帧直接渲染到独立窗口，避免 `UI` 线程阻塞
- ​**双缓冲机制**​：使用前后台缓冲区交替渲染，提升播放流畅性
- ​**生命周期控制**​：在 `surfaceCreated` 中初始化播放器，在 `surfaceDestroyed` 中释放资源，防止内存泄漏

##### 2.代码示例
```java
TextureView textureView = findViewById(R.id.textureView);
textureView.setSurfaceTextureListener(new TextureView.SurfaceTextureListener() {
    @Override
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        IjkMediaPlayer player = new IjkMediaPlayer();
        player.setSurface(new Surface(surface));  // 绑定 SurfaceTexture
        player.setDataSource("video_url");
        player.prepareAsync();
    }
    // 其他回调方法（onSurfaceTextureSizeChanged, onSurfaceTextureDestroyed）
});
```

##### 3.优缺点
- **优点**​：
    - 性能高，适合全屏播放和硬解码场景
    - 内存占用低，资源管理简单。
- ​**缺点**​：
    - 无法与其他 View 叠加或动态调整布局
    - 全屏切换需重建 `SurfaceView`，可能引发短暂黑屏

#### 三、`IjkPlayer` 中 `TextureView`的应用

##### 1.实现原理
- **集成 View 层级**​：通过 `SurfaceTextureListener` 监听 `SurfaceTexture` 的创建与销毁，动态绑定视频渲染目标
- ​**硬件加速支持**​：利用 `OpenGL ES` 直接渲染到 `SurfaceTexture`，支持旋转、缩放等变换
- ​**动态布局**​：可嵌入 `FrameLayout` 或 `RelativeLayout`，与其他控件（如弹幕、封面）共存

##### 2.代码示例
```java
TextureView textureView = findViewById(R.id.textureView);
textureView.setSurfaceTextureListener(new TextureView.SurfaceTextureListener() {
    @Override
    public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
        IjkMediaPlayer player = new IjkMediaPlayer();
        player.setSurface(new Surface(surface));  // 绑定 SurfaceTexture
        player.setDataSource("video_url");
        player.prepareAsync();
    }
    // 其他回调方法（onSurfaceTextureSizeChanged, onSurfaceTextureDestroyed）
});
```

##### 3.优缺点
- **优点**​：
    - 支持动态布局调整（如全屏切换、弹幕叠加）
    - 与 Android 动画系统兼容，可实现旋转、缩放等效果
- ​**缺点**​：
    - 渲染性能略低于 `SurfaceView`，复杂场景可能引发卡顿
    - 需处理 Surface 生命周期，代码复杂度较高

#### 四、`IjkPlayer`的实际选择与优化
##### **1. 默认选择**​
- ​`SurfaceView`​：`IjkPlayer` 默认推荐使用 `SurfaceView`，因其性能优势明显，尤其适合直播流、长视频等场景
- ​`TextureView`​：在需要动态布局（如列表播放、弹幕）时，需手动集成 `TextureView` 并适配渲染逻辑
##### ​**2. 性能优化**​
- ​**双缓冲与线程管理**​：确保视频解码与渲染线程分离，避免 `UI` 卡顿
- ​**硬件解码**​：通过 `IjkMediaPlayer.setOption("mediacodec", 1)` 启用 `MediaCodec` 硬解，降低 CPU 负载
- ​**内存回收**​：在 `surfaceDestroyed` 或 `onSurfaceTextureDestroyed` 中释放 `MediaPlayer` 资源，防止内存泄漏
##### ​**3. 全屏切换实现**​
- ​`SurfaceView` 方案**​：通过隐藏原 `SurfaceView` 并创建新的全屏 `SurfaceView`，结合动画实现切换
- ​`TextureView` 方案**​：直接调整 `TextureView` 的布局参数（如宽高设为屏幕尺寸），无需重建对象
