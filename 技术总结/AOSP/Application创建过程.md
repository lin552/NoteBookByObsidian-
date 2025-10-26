---
创建时间: "2025-08-20 15:21:27"
作者: wangxiaoming
tags:
---
#### 一、Android Application的创建过程

##### 1. `ActivityThread` 绑定 `AMS`
应用进程启动后，`ActivityThread`实例通过 `attach(false)`方法与 `AMS`（Activity Manager Service）建立 Binder 通信。此步骤的核心是告知 `AMS`当前进程已准备好接收组件任务”，为后续 Application 创建做铺垫。
##### ​2. `AMS` 触发 Application 创建**​
`AMS` 收到 `ActivityThread`的绑定请求后，会从 `AndroidManifest.xml`中读取应用声明的 `Application`类（若未自定义，使用系统默认的 `Application`）。随后，AMS 通过 ​**反射机制**​ 触发该类的实例化。
##### ​**3. 反射调用无参构造方法创建实例**​
系统通过反射调用 `Application`类的 ​**无参构造方法**​ 创建实例。这是关键限制：
- ​**必须存在无参构造**​：反射实例化的规则是优先调用无参构造；若自定义 `Application`时添加了带参构造（如 `public MyApplication(Context context)`），系统无法传递参数，反射会失败（抛出 `InstantiationException`），导致应用无法启动。
- **父类约束**​：`Application`继承自 `ContextWrapper`，其构造方法需 `Context`参数，但系统通过反射绕过此限制，直接调用无参构造（后续通过 `attachBaseContext()`绑定 `Context`）。
##### ​**4. 调用 `attachBaseContext(Context)` 绑定上下文​
实例创建后，系统立即调用 `attachBaseContext(Context base)`方法，将进程的 `ContextImpl`（应用级上下文）绑定到 `Application`实例（即 `base`参数）。此时：
- `Application`获得基础上下文，可访问 `Context`基础功能（如 `getPackageName()`）。
- 但资源（如 `getResources()`）、系统服务（如 `getSystemService()`）尚未完全初始化，仅适合**早期配置**​（如多语言设置、`MultiDex` 安装）。
##### ​**5. 调用 `onCreate()` 完成初始化**​
`attachBaseContext()`执行完成后，系统调用 `onCreate()`方法。此时：
- `Context`已完全初始化（资源、系统服务、组件管理器就绪）。
- 适合执行**全局初始化操作**​（如第三方 `SDK` 初始化、数据库创建、全局变量配置）。

#### 二、为什么不能用构造参数创建Application?
核心原因与系统实例化机制和生命周期执行时机强相关：
##### **1. 系统反射仅支持无参构造**​
AMS 通过反射创建 `Application`实例时，​**必须调用无参构造方法**​（反射规则）。若自定义 `Application`包含带参构造（即使编译器自动生成的无参构造被覆盖），系统无法传递参数，反射失败，应用启动崩溃。
##### ​**2. 构造方法执行时 Context 未初始化**​
即使允许带参构造，`Application`的构造方法执行时：
- `attachBaseContext(Context)`未调用，`Context`未绑定（`mBase`为 `null`）。
- 无法访问资源、系统服务等依赖 `Context`的功能，传递 `Context`参数无实际意义。
##### ​**3. 框架设计强制初始化在生命周期方法中**​
Android 规定，`Application`的初始化逻辑需在 `attachBaseContext()`（早期配置）和 `onCreate()`（完整初始化）中完成，构造方法仅用于创建实例，不参与实际初始化。