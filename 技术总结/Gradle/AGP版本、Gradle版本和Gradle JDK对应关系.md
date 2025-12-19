---
创建时间: "2025-11-05 14:23:28"
作者: wangxiaoming
tags:
---
#### 一、核心版本对应关系
表格仅为参考

| AGP 版本              | 所需 Gradle 版本     | 所需 JDK 版本                             |
| ------------------- | ---------------- | ------------------------------------- |
| ​**AGP 8.3+​**​     | Gradle 8.4+      | ​**JDK 17+​**​ (强制要求)                 |
| ​**AGP 8.0 - 8.2**​ | Gradle 8.0 - 8.3 | ​**JDK 17+​**​ (强制要求)                 |
| ​**AGP 7.4 - 7.5**​ | Gradle 7.5+      | ​**JDK 11**​ (官方推荐)，可在 JDK 8-17 范围内使用 |
| ​**AGP 7.0 - 7.3**​ | Gradle 7.0+      | ​**JDK 11**​ (官方推荐)，可在 JDK 8-11 范围内使用 |
| ​**AGP 4.2+​**​     | Gradle 6.7.1+    | ​**JDK 8+​**​ (最低要求)                  |
https://developer.android.com/build/releases/gradle-plugin?hl=zh-cn 参考公共文档
#### 二、配置AGP版本
- ​**配置 AGP 版本**​：在项目根目录（Project Root）的 `build.gradle`文件中的 `dependencies`块内进行配置。
```gradle
dependencies {
    classpath "com.android.tools.build:gradle:8.2.0" // 此处指定 AGP 版本
}
```
- ​**配置 Gradle 版本**​：在 `gradle/wrapper/gradle-wrapper.properties`文件中，通过 `distributionUrl`属性进行配置。
```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-bin.zip
```
- ​**配置 Gradle JDK**​：在 Android Studio 中，通过 ​**File > Project Structure > Project**​ 界面，在 ​**Gradle JDK**​ 下拉选项中进行选择。