---
创建时间: 2025-06-13 19:30:06
作者: wangxiaoming
tags:
  - OOM
---
在Android开发中，内存溢出（`OOM`，Out Of Memory）是指应用申请的内存超过了系统为其分配的最大内存限制，导致应用崩溃。以下是**Android中最常见的`OOM`场景**及**针对性解决方法**，结合原理与实战经验总结：

| **场景分类**​        | ​**场景描述**​                                                                                             | ​**核心原因**​                                            | ​**具体表现**​                                                                          | ​**解决方法**​                                                                                                                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ​**图片加载不当**​     | 加载高分辨率图片（如相机原图、高清海报）未压缩直接加载到内存。                                                                        | Bitmap内存占用公式：`宽×高×像素字节数`（如ARGB_8888占4字节），大尺寸图片易超内存限制。 | 加载多张大图后内存激增，触发OOM崩溃（日志提示`OutOfMemoryError: Bitmap size exceeds VM budget`）。         | - 压缩尺寸：通过`BitmapFactory.Options.inSampleSize`按比例缩小（如`inSampleSize=2`降为1/2尺寸）；  <br>- 限制加载尺寸：根据ImageView尺寸用`override(width, height)`指定加载大小；  <br>- 使用图片库（Glide/Picasso）自动处理压缩与缓存。                                 |
| ​**大对象未释放**​     | 循环中频繁创建临时对象（如`String +`拼接）；长生命周期对象（单例/静态变量）持有大对象（如`Bitmap`、`List`）；大数组/集合未清理。                          | 频繁创建大对象导致内存快速耗尽；长生命周期对象阻止GC回收大对象，累积后超内存限制。            | 内存占用持续增长，频繁GC仍无法释放，最终OOM（日志提示`GC overhead limit exceeded`）。                         | - 避免内存抖动：循环中用`StringBuilder`代替`String +`；  <br>- 释放大对象引用：长生命周期对象用`WeakReference`持有；  <br>- 清理集合：静态集合存储大对象后及时`clear()`；  <br>- 复用对象池（如`SparseArray`、`LruCache`）。                                                  |
| ​**内存泄漏导致累积溢出**​ | Activity/Fragment被长生命周期对象（静态变量、单例、匿名内部类）强引用无法回收；未关闭的资源（如`Cursor`）隐式持有Context；第三方库未释放监听器。               | 短生命周期对象（Activity）被长生命周期对象强引用，GC无法回收，反复创建后内存累积超限。      | 页面反复打开/关闭后内存持续增长，最终OOM（日志无明确提示，但内存曲线呈上升趋势）。                                         | - 弱引用替代强引用：静态变量用`WeakReference<Activity>`；  <br>- 移除回调：匿名内部类改用静态内部类+弱引用，在`onDestroy()`取消注册（如`Handler.removeCallbacks()`）；  <br>- 及时关闭资源：`Cursor`/`InputStream`在`finally`块关闭；  <br>- 检测工具定位：LeakCanary抓取堆转储分析泄漏链。 |
| ​**资源未释放或过度使用**​ | 未释放`Bitmap`（未调用`recycle()`）；重复加载同一大文件到内存；大文件一次性读取到`byte[]`。                                            | 资源未释放导致内存持续占用；大文件一次性加载超出内存容量。                         | 内存占用居高不下（如`Bitmap`未释放）；读取大文件时直接OOM（日志提示`Failed to allocate a ... byte allocation`）。 | - 释放`Bitmap`：API 29前手动调用`bitmap.recycle()`并置空；  <br>- 分块处理大文件：分段读取（如每次1MB），避免一次性加载；  <br>- 复用资源：重复使用的资源（如图片）缓存到内存池（如`LruCache`）。                                                                                 |
| ​**其他场景**​       | RecyclerView未优化（`onBindViewHolder`重复创建对象）；Activity主题设置大背景（`android:windowBackground`为高清图）；多线程并发加载同一资源。 | 资源未合理优化导致内存浪费；多线程竞争加载资源导致重复分配内存。                      | 滑动卡顿（RecyclerView）；启动时内存激增（主题背景过大）；多线程加载同一资源时内存峰值过高。                                | - RecyclerView优化：使用`ViewHolder`复用ItemView，避免`onBindViewHolder`创建新对象；  <br>- 移除冗余背景：检查布局和主题的`background`属性，避免重复设置；  <br>- 线程同步：加载资源时加锁（`synchronized`）或使用单例加载器。                                                   |

#### 一、图片加载不当导致的`OOM`
**场景**​：加载高分辨率图片（如相机拍摄的原图、高清海报）时，未做压缩直接加载到内存，导致Bitmap占用内存过大。  
​**原理**​：Bitmap的内存占用计算公式为 `宽 × 高 × 每个像素占用的字节数`（如`ARGB_8888`格式每个像素占4字节）。例如，一张1080×1920的`ARGB_8888`图片占用约`8MB`内存（1080×1920×4≈8,294,400字节≈`8MB`），若同时加载多张此类图片，易超出内存限制。

**解决方法**​：
1. ​**压缩图片尺寸**​：通过`BitmapFactory.Options`的`inSampleSize`参数按比例缩小图片（如`inSampleSize=2`则宽高各缩为1/2，内存占用降为1/4）。
```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inSampleSize = 2; // 缩放比例（2的幂次方效果最佳）
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.large_image, options);
```
2. ​**限制图片分辨率**​：根据`ImageView`的尺寸加载匹配的图片（如`ImageView`宽高为`200dp×200dp`，只需加载200×200的图片）。
3. ​**使用图片加载库**​（Glide/Picasso）：自动处理压缩、缓存和内存管理，避免手动加载的风险。

#### 二、大对象未释放或频繁创建
**场景**​：
- 循环中频繁创建临时对象（如`String`拼接用`+`号，每次生成新对象）；
- 长生命周期对象（如单例、静态变量）持有大对象（如`Bitmap`、`List`）的强引用，导致无法回收；
- 大数组/集合未及时清理（如静态`List`存储大量数据但未释放）。

​**原理**​：Java堆内存有限（受设备RAM限制，如低端机可能仅`1GB`），频繁创建大对象会导致内存快速耗尽；长生命周期对象持有大对象引用会阻止`GC`回收，累积后超出内存限制。

**解决方法**​：
1. **避免内存抖动**​：用`StringBuilder`代替`String +`拼接（尤其在循环中）；
```java
// 错误示例（内存抖动）
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // 每次生成新String对象
}

// 正确示例（复用对象）
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```
2. ​**释放大对象引用**​：长生命周期对象（如单例）持有大对象时，使用`WeakReference`（弱引用，`GC`时自动回收）；
```java
public class Singleton {
    private static Singleton instance;
    private WeakReference<Bitmap> bitmapRef; // 弱引用持有Bitmap
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public void setBitmap(Bitmap bitmap) {
        this.bitmapRef = new WeakReference<>(bitmap);
    }
    
    public Bitmap getBitmap() {
        return bitmapRef != null ? bitmapRef.get() : null;
    }
}
```
3. **及时清理集合**​：静态集合（如`static List`）存储大对象后，需在适当时候（如`onDestroy()`）调用`clear()`释放；
4. ​**复用对象池**​：高频创建的对象（如`Message`、`Bitmap`）使用对象池复用（如`SparseArray`、`LruCache`）。

#### 三、内存泄漏导致累积溢出
**场景**​：Activity/Fragment等短生命周期对象被长生命周期对象（如静态变量、单例、匿名内部类）强引用，导致其无法被`GC`回收，反复创建后内存累积超出限制。

​**常见泄漏场景**​：

- 静态变量持有Activity（如`static Context mContext = activity`）；
- 匿名内部类/回调未解除（如`Handler`持有Activity引用，未在`onDestroy()`移除消息）；
- 未关闭的资源（如`Cursor`、`InputStream`）隐式持有Context；
- 第三方库未正确释放（如广告`SDK`、推送`SDK`的全局监听器）。

**解决方法**​：
1. ​**避免静态变量强引用Activity**​：改用`WeakReference`或通过`Application`上下文（`getApplicationContext()`）获取全局上下文；
2. ​**移除匿名内部类引用**​：
    - 使用静态内部类+弱引用（如`Handler`）；
    - 在`onDestroy()`中取消注册（如`unregisterReceiver()`、`removeCallbacks()`）；
```kotlin
// 正确使用Handler（避免泄漏）
private val handler = object : Handler(Looper.getMainLooper()) {
    override fun handleMessage(msg: Message) {
        // 处理消息
    }
}

override fun onDestroy() {
    super.onDestroy()
    handler.removeCallbacksAndMessages(null) // 移除所有消息
}
```
3. ​**及时关闭资源**​：在`finally`块中关闭`Cursor`、`InputStream`等（API 29+可用`use`自动关闭）；
```java
Cursor cursor = null;
try {
    cursor = db.rawQuery("SELECT * FROM table", null);
    // 使用cursor
} finally {
    if (cursor != null) cursor.close(); // 确保关闭
}
```
4. ​**检测工具定位泄漏**​：使用`LeakCanary`或Android Studio的`Memory Profiler`抓取堆转储（Heap Dump），分析`GC Roots`找到泄漏链。

#### 四、资源未释放或过度使用
**场景**​：
- 未释放`Bitmap`（`Bitmap.recycle()`）；
- 重复加载同一资源（如多次读取大文件到内存）；
- 大文件直接加载到内存（如读取`100MB`的日志文件到`byte[]`）。

**解决方法**​：
1. ​**释放Bitmap资源**​（API 29前有效，API 29后系统自动回收）：
```java
if (bitmap != null && !bitmap.isRecycled()) {
    bitmap.recycle(); // 释放Native内存
    bitmap = null; // 帮助GC回收Java对象
}
```
1. ​**分块处理大文件**​：读取大文件时分段加载（如每次读取`1MB`），避免一次性加载到内存；
```java
try (FileInputStream fis = new FileInputStream(file)) {
    byte[] buffer = new byte[1024 * 1024]; // 1MB缓冲区
    int len;
    while ((len = fis.read(buffer)) != -1) {
        // 处理当前buffer（1MB）
    }
}
```

#### 五、其他场景

**场景**​：
- 大数据量的`RecyclerView`未优化（如`onBindViewHolder`中重复创建对象）；
- `Activity`主题设置过大背景（如`android:windowBackground`为高清图）；
- 多线程并发加载资源（如多个线程同时加载图片到同一`ImageView`）。

​**解决方法**​：
- `RecyclerView`优化：使用`ViewHolder`复用`ItemView`，避免在`onBindViewHolder`中创建新对象；
- 移除冗余背景：检查`Activity`主题和布局文件的`background`属性，避免重复设置；
- 线程同步：加载资源时加锁（如`synchronized`）或使用单例加载器，避免重复加载。