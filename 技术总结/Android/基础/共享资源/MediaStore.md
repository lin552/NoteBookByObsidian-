---
创建时间: 2025-05-24 13:59:09
作者: wangxiaoming
tags:
  - ContentProvider
---
#### 一、基本概念
1. `MediaStore`是什么？有什么作用？​**​
    - ​**回答**​：`MediaStore`是`Android`系统提供的**媒体文件共享数据库**​（属于`ContentProvider`的子类），用于统一管理设备上的媒体文件（图片、视频、音频）。其他应用可通过`ContentResolver`访问这些文件，无需直接操作文件路径。
    - ​**核心特点**​：
        - ​**自动索引**​：系统会自动扫描并记录媒体文件的元数据（如文件名、类型、时间戳）。
        - ​**跨应用共享**​：通过统一的`URI`访问媒体文件，支持权限控制。
        - ​**适配Scoped Storage**​：Android 10+中，应用无需`READ_EXTERNAL_STORAGE`权限即可访问公共媒体文件。
2. ​`MediaStore`与`FileProvider`的区别？​**​
    - ​**回答**​：
        - ​`MediaStore`​：专为媒体文件设计，自动索引并提供`URI`访问，支持跨应用共享。
        - ​`FileProvider`​：通用文件共享组件，适用于任意文件，但需手动配置路径和`URI`。
#### 二、核心操作与`URI`机制
​   3. ​如何通过`MediaStore`查询媒体文件？​**​
    - ​**回答**​：使用`ContentResolver.query()`方法，指定`URI`和投影（Projection）：
```java
Uri uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
String[] projection = {MediaStore.Images.Media._ID, MediaStore.Images.Media.DISPLAY_NAME};
Cursor cursor = getContentResolver().query(uri, projection, null, null, null);
while (cursor.moveToNext()) {
    String id = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID));
    String name = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME));
}
cursor.close();
```
4. ​如何插入一张图片到`MediaStore`？​​
    - ​**回答**​：通过`ContentResolver.insert()`生成`URI`，写入内容：
```java
ContentValues values = new ContentValues();
values.put(MediaStore.Images.Media.DISPLAY_NAME, "photo.jpg");
values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
values.put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES + "/MyApp/");

Uri uri = getContentResolver().insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
try (OutputStream os = getContentResolver().openOutputStream(uri)) {
    // 写入图片数据到os
}
```
4. ​`MediaStore`的`URI`结构是怎样的？​**​
    - ​**回答**​：`URI`格式为`content://media/external/images/media/123`，其中：
        - `external`：外部存储。
        - `images`：媒体类型（图片、视频、音频）。
        - `123`：数据库中的唯一ID。
    - ​**路径映射**​：通过`RELATIVE_PATH`定义文件在存储中的位置（如`Environment.DIRECTORY_PICTURES`对应`Pictures/`目录）。
#### 三、权限与版本适配
6. ​访问`MediaStore`需要哪些权限？​**​
    - ​**回答**​：
        - ​**Android 9及以下**​：需声明`READ_EXTERNAL_STORAGE`或`WRITE_EXTERNAL_STORAGE`。
        - ​**Android 10+​**​：通过`MediaStore` API访问公共媒体文件无需权限，但写入需`WRITE_EXTERNAL_STORAGE`（部分设备可能豁免）。
        - ​**Android 11+​**​：使用`MANAGE_EXTERNAL_STORAGE`可访问所有文件，但需用户手动授权。
7. ​**如何适配Scoped Storage？​**​
    - ​**回答**​：
        - 使用`MediaStore API`操作公共媒体文件，无需请求存储权限。
        - 写入文件时，通过`RELATIVE_PATH`指定目录（如`Environment.DIRECTORY_DOCUMENTS`）。
```java
values.put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_DOCUMENTS + "/MyApp/");
```
#### 四、应用场景与问题
8. **常见的使用场景有哪些？​**​
    - ​**回答**​：
        - 相册应用读取设备图片/视频。
        - 保存用户生成的图片到公共相册。
        - 批量处理媒体文件（如批量打印）。
9. ​**如何监听媒体文件变化？​**​
    - ​**回答**​：注册`ContentObserver`监听`MediaStore`的`URI`：
        ```java
        ContentObserver observer = new ContentObserver(new Handler()) {
            @Override
            public void onChange(boolean selfChange) {
                // 重新查询数据
            }
        };
        getContentResolver().registerContentObserver(
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
            true, 
            observer
        );
        ```
10. ​**为何部分设备无法通过`MediaStore`访问文件？​**​
    - ​**回答**​：
        - 文件未正确插入数据库（如缺少`DISPLAY_NAME`或`MIME_TYPE`）。
        - 设备厂商修改了`MediaStore`的默认路径（需通过`RELATIVE_PATH`适配）。
#### 五、高频陷阱题
11. ​**插入文件后为何其他应用无法看到？​**​
    - ​**回答**​：
        - 未调用`notifyChange(uri, null)`通知系统更新索引。
        - 文件未写入完成就关闭流（需确保`OutputStream`正确关闭）。
12. ​**如何获取文件的真实路径？​**​
    - ​**回答**​：避免直接获取路径，应始终通过`URI`操作。若必须路径，使用`FileDescriptor`或`ContentResolver.openInputStream()`。
#### 六、进阶问题
13. ​**如何通过`MediaStore`访问非媒体文件？​**​
    - ​**回答**​：`MediaStore`仅支持媒体文件。非媒体文件需使用`Storage Access Framework`（`SAF`）或`FileProvider`。
14. ​`MediaStore`与`SAF`（Storage Access Framework）的关系？​**​
    - ​**回答**​：
        - ​`MediaStore`​：适合访问已存在的媒体文件。
        - ​`SAF`​：允许用户选择任意文件或目录，返回`URI`，适合保存或读取非媒体文件。