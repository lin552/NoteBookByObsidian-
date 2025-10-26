---
创建时间: "2025-06-16 01:06:25"
作者: wangxiaoming
tags:
---
#### 一、核心概念与设计目标

##### 1.Scoped Storage 是什么？
- ​**定义**​：Android 10（API 29）引入的存储机制，通过**限制应用对外部存储的访问范围**，增强用户隐私和数据安全。
- ​**核心目标**​：
    - 防止应用滥用共享存储空间（如 SD 卡），避免隐私泄露。
    - 简化用户对文件的管理（如清理应用专属文件）。
##### 2.核心变化
- **私有目录隔离**​：每个应用在外部存储拥有独立目录（`Android/data/<包名>/`），其他应用无法直接访问
- ​**共享目录限制**​：直接访问公共目录（如 `DCIM`、`Downloads`）需通过 `MediaStore` 或 `Storage Access Framework (SAF)`
- ​**权限细化**​：Android 13+ 将 `WRITE_EXTERNAL_STORAGE` 拆分为 `IMAGE`、`AUDIO`、`VIDEO` 等细粒度权限

#### 二、适配方法与高频代码
##### 1.媒体文件操作（图片/视频/音频）
 **使用 `MediaStore` 插入文件**​（无需权限，自动归属媒体库）：\
```java
ContentValues values = new ContentValues();
values.put(MediaStore.Images.Media.DISPLAY_NAME, "photo.jpg");
values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
values.put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES);
ContentResolver resolver = getContentResolver();
Uri uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
// 写入文件流
try (OutputStream os = resolver.openOutputStream(uri)) {
    // 写入数据...
}
```
##### 2.非媒体文件操作（PDF/DOC等）
**通过 `Storage Access Framework (SAF)` 请求用户授权**​：
```java
Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
intent.addCategory(Intent.CATEGORY_OPENABLE);
intent.setType("*/*");
startActivityForResult(intent, REQUEST_CODE_OPEN_FILE);
```

#### 3.访问应用私有目录
​**直接读写（无需权限）​**​：
```java
File privateDir = getExternalFilesDir("docs"); // Android 10+ 仍可访问
File file = new File(privateDir, "data.txt");
```

#### 三、权限管理考点
##### 1.动态权限申请
**媒体文件访问**​：
```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />
```

```java
if (ContextCompat.checkSelfPermission(this, READ_MEDIA_IMAGES) != PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[]{READ_MEDIA_IMAGES}, REQUEST_CODE);
}
```

##### 2.`MANAGE_EXTERNAL_STORAGE` 权限
- **适用场景**​：文件管理类应用需全局访问存储（如文件浏览器）。
- ​**申请方式**​：
```java
if (!Environment.isExternalStorageManager()) {
    Intent intent = new Intent(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
    intent.setData(Uri.parse("package:" + getPackageName()));
    startActivity(intent);
}
```
​**限制**​：需 Google Play 审核，仅限必要场景使用

#### 四、兼容性处理
##### 1.Android 10兼容（临时关闭 Scoped Storage）
在 `AndroidMainfest.xml` 中声明
```xml
<application
    android:requestLegacyExternalStorage="true"
    ...>
</application>
```
注意：Android 11+ 强制启用 Scoped Storage，此方法失效

##### 2.旧文件路径访问
​**通过 `MediaStore` 查询历史文件**​：
```java
Cursor cursor = getContentResolver().query(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    new String[]{MediaStore.Images.Media._ID},
    null, null, null
);
```

##### 五、高频面试题
##### 1. ​**Scoped Storage 的引入原因？​**​
- ​**隐私保护**​：防止应用滥用共享存储空间，避免用户数据泄露
- ​**简化管理**​：用户可更直观控制应用文件（如卸载应用时自动清理专属目录）
##### 2. ​**Android 10 和 Android 11 的差异？​**​
- ​**Android 10**​：允许通过 `requestLegacyExternalStorage` 临时禁用 Scoped Storage。
- ​**Android 11**​：强制启用，且需 `MANAGE_EXTERNAL_STORAGE` 权限才能访问公共目录所有文件
##### 3. ​**如何适配旧版本文件路径？​**​
- ​**使用 `MediaStore` 或 `SAF`**​：替代直接文件路径操作。
- ​**示例**​：通过 `MediaStore` 查询历史图片：
 ```java
   Cursor cursor = getContentResolver().query(
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        null, null, null, null
    );
```
##### 4. ​**`MANAGE_EXTERNAL_STORAGE` 的限制？​**​
- ​**仅限必要场景**​：如文件管理、备份工具。
- ​**需用户手动授权**​：在系统设置中开启，且 Google Play 会审核合理性

#### 六、底层原理与扩展
##### 1. ​**存储沙盒机制**​
- ​**应用私有目录**​：`Android/data/<包名>/`，卸载应用时自动删除。
- ​**共享媒体库**​：通过 `MediaStore` 索引，支持跨应用访问（如相册显示图片）
##### 2. ​**文件分类与元数据**​
- ​**自动标记**​：系统根据文件类型（图片/视频/音频）分类存储，并关联元数据（如拍摄时间、地理位置）
##### 3. ​**性能优化**​
- ​`F2FS` 文件系统**​：减少写入放大，延长设备寿命（Android 12+ 默认启用）
