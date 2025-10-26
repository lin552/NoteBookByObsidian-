---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Bitmap
---
Bitmap 是 Android 中处理图像的核心类，用于存储和操作位图数据。其核心考点围绕**内存管理**、**加载优化**、**绘制优化**及**常见陷阱**展开，以下是系统梳理：

#### 一、Bitmap基础概念

`Bitmap`的内存主要由两部分构成：
1. ​**像素数据（Pixel Data）​**​：存储图像的颜色信息（如 `RGB`、`Alpha` 通道），占内存的 90% 以上。
2. **元数据（Metadata）​**​：包括宽度、高度、颜色配置（`Config`）、缩放类型等描述信息，占内存极小（通常可忽略）。

##### 1. 核心特性​
- ​**像素存储**​：Bitmap 以二维数组形式存储像素数据，每个像素的颜色值由配置（`Config`）决定。
- ​**内存占用**​：内存大小 = 宽度 × 高度 × 每像素字节数（由 `Config` 决定）。

|配置|描述|每个像素字节数|典型场景|
|---|---|---|---|
|`ARGB_8888`|32 位真彩色（Alpha-Red-Green-Blue 各 8 位）|4|高质量图片（默认配置）|
|`RGB_565`|16 位彩色（Red-5 位，Green-6 位，Blue-5 位，无 Alpha）|2|无透明需求的图片（如游戏）|
|`ARGB_4444`|16 位彩色（Alpha-4 位，Red-4 位，Green-4 位，Blue-4 位）|2|已过时（精度低，不推荐使用）|
|`ALPHA_8`|仅存储 Alpha 通道（透明度），无颜色信息|1|仅需要透明度的遮罩图|
|`RGBA_F16`|64 位浮点彩色（每个通道 16 位，支持 HDR）|8|高动态范围图像（如专业摄影）|

- ​**可变性**​：默认不可变（`isMutable()` 返回 `false`），需通过 `copy()` 或 `Bitmap.createBitmap()` 创建可变副本。
##### ​2. 加载与创建​
- ​**从资源加载**​：
    ```java
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image);
    ```
- ​**从文件加载**​：
    ```java
    Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/image.jpg");
    ```
- ​**从字节数组加载**​：
    ```java
    byte[] data = ...; // 网络或本地读取的字节数据
    Bitmap bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);
    ```
##### 3.Bitmap内存的特殊机制（Android版本差异）
###### ①`Android3.0 （API11)`前的内存分配
- 像素数据存储在 ​**Native 内存**​（非 Java 堆），由 C/C++ 层管理。
- Java 层的 `Bitmap`对象仅持有 Native 内存的指针，垃圾回收（`GC`）无法直接回收 Native 内存。
- 需手动调用 `recycle()`释放 Native 内存（否则可能导致内存泄漏）。
###### ② `Android3.0(API11) - Android7.0(API24)`
- 像素数据迁移到 ​**Java 堆**​（`Dalvik/ART `堆），由 `GC`自动管理。
- 优势：避免 Native 内存泄漏，`GC` 自动回收。
- 劣势：大图片可能直接导致 Java 堆 `OOM`（因 Java 堆大小受限于设备内存）。
###### ③ `Android8.0(API 26)`及以后
- 引入 ​`Ashmem`（匿名共享内存）​​ 优化：大尺寸 `Bitmap`的像素数据存储在 `Ashmem` 中，而非 Java 堆或纯 Native 内存。
- 特点：
    - 内存可被多个进程共享（如通过 `Bitmap`传递到其他进程）。
    - 当 `Bitmap`不再被引用时，`Ashmem` 会自动回收内存（减少手动管理成本）。
    - 跨进程传递时需通过 `Parcel`序列化，可能产生额外开销。
#### 二、内存管理与`OOM`避免
Bitmap 是 Android 应用中**内存占用大户**，不当使用易导致 `OOM`（内存溢出）。核心优化手段如下：
##### ​1. 内存计算与采样率（`inSampleSize`）​​
- ​**采样率（`inSampleSize`）​**​：通过 `BitmapFactory.Options` 设置，用于按比例缩小图片尺寸，减少内存占用。
    - 规则：`inSampleSize` 为 2 的幂次（如 1、2、4），实际采样为原图的 `1/inSampleSize`。
    - 计算方法：根据目标尺寸与原图尺寸的比例，取最大的 2 的幂次。
    ​**示例：计算合适的 `inSampleSize`**​
    ```java
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true; // 仅获取图片尺寸，不加载内存
    BitmapFactory.decodeFile(path, options);
    
    // 目标尺寸（如 ImageView 的宽高）
    int targetWidth = 200;
    int targetHeight = 200;
    
    // 计算宽高比
    int widthRatio = options.outWidth / targetWidth;
    int heightRatio = options.outHeight / targetHeight;
    options.inSampleSize = Math.max(widthRatio, heightRatio); // 取最大比例
    
    options.inJustDecodeBounds = false; // 加载缩小后的图片
    Bitmap bitmap = BitmapFactory.decodeFile(path, options);
    ```

##### ​2. 避免重复解码​
- ​**复用 `Bitmap`​：通过 `BitmapFactory.Options.inMutable` 和 `Bitmap.copy()` 复用已有 Bitmap 的内存。
- ​**使用 `BitmapPool`​（如 Glide 内部实现）：缓存可复用的 Bitmap 内存块，减少 `GC` 压力。

##### ​**3. 回收与内存释放**​
- `recycle()` 方法**​（API 29 前）：手动释放 Bitmap 内存（标记为可回收）。
    - 注意：`recycle()` 后 Bitmap 不可再使用，否则会抛出 `IllegalStateException`。
    - 弊端：API 29 后被标记为过时，推荐通过弱引用（`WeakReference`）或缓存框架（如 Glide）自动管理。

#### 三、绘制优化与高级特性
##### ​1. Canvas 绘制优化​
- ​**避免重复创建 Bitmap**​：在 `onDraw(Canvas)` 中避免重复解码 Bitmap（应在 `onCreate` 或 `onLoad` 中提前加载）。
- ​使用 `BitmapShader`：通过 `Paint.setShader(BitmapShader)` 实现图片的平铺、缩放等效果，减少重复绘制。
    ```java
    Paint paint = new Paint();
    BitmapShader shader = new BitmapShader(bitmap, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT);
    paint.setShader(shader);
    canvas.drawCircle(100, 100, 50, paint); // 绘制圆形图片
    ```

##### ​**2. 硬件加速与兼容性**​
- ​**硬件加速**​：Android 3.0（API 11）后默认开启，Bitmap 绘制由 GPU 处理，提升性能。
- ​**兼容低版本**​：部分操作（如 `Bitmap.setHasAlpha()`）在低版本可能失效，需做版本判断。

##### ​**3. 图片格式选择**​
- ​`JPEG`​：无透明度，适合照片类图片（压缩率高，文件小）。
- ​`PNG`：支持透明度，适合图标、线条图（无损压缩，文件较大）。
- ​`WebP`：Google 推出的格式，支持有损/无损压缩，文件比 `JPEG/PNG` 更小（需 API 14+）。

#### 四、高频面试题
1. ​**Bitmap 的内存占用如何计算？不同 `Config` 的区别是什么？​**​  
    内存 = 宽 × 高 × 每像素字节数。`ARGB_8888` 占 4 字节/像素（支持透明度），`RGB_565` 占 2 字节/像素（无透明度）。
    
2. ​**如何避免 Bitmap 导致的 `OOM`？​**​
    - 使用 `BitmapFactory.Options.inSampleSize` 采样缩小图片。
    - 复用 Bitmap 内存（如 `BitmapPool`）。
    - 及时回收不再使用的 Bitmap（`recycle()`，API 29 前）。
    
3. ​`inJustDecodeBounds` 的作用是什么？如何利用它优化加载？​**​  
    `inJustDecodeBounds = true` 时，`decodeXXX()` 仅解析图片尺寸（不加载内存），可用于计算采样率，避免直接加载大图导致 `OOM`。
    
4. ​Bitmap 的 `recycle()` 方法有什么作用？需要注意什么？​**​  
    标记 Bitmap 内存为可回收，释放底层内存。注意：调用后 Bitmap 不可再使用，否则抛出异常；API 29 后已过时，推荐使用缓存框架管理。
    
5. ​**如何选择 Bitmap 的 `Config`？​**​
    - 需要透明度（如 `PNG`）→ `ARGB_8888`。
    - 不需要透明度且颜色简单（如图标）→ `RGB_565`。
    - 需要高质量（如照片）→ `ARGB_8888`。
#### 五、源码与实战技巧
##### 1. Bitmap 的内部存储​
Bitmap 的像素数据存储在 `mBuffer`（`byte[]` 或 `int[]`）中，通过 `rowBytes`（每行字节数）计算内存偏移。

##### ​2. Glide 中的 Bitmap 优化​
Glide 底层通过 `BitmapPool` 复用 Bitmap 内存，结合`LRU`缓存策略，避免频繁创建/销毁 Bitmap。核心逻辑：
- 从 `BitmapPool` 申请可用 Bitmap（匹配尺寸和配置）。
- 加载完成后，若 `BitmapPool` 有空间则回收，否则释放。