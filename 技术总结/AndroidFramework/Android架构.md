---
创建时间: "2025-08-25 14:26:51"
作者: wangxiaoming
tags:
---
![[Pasted image 20250825142652.png]]
1. **Android 应用（Android Apps）**
    完全使用 Android API 开发的应用。在某些情况下，设备制造商可能希望预安装 Android 应用以支持设备的核心功能。
2. **特权应用（Privileged Apps）**
    使用 Android 和系统 API 组合创建的应用。这些应用必须作为特权应用预安装在设备上。
3. **设备制造商应用（Device Manufacturer Apps）**
    结合使用 Android API、系统 API 并直接访问 Android 框架实现而创建的应用。由于设备制造商可能会直接访问 Android 框架中的不稳定的 API，因此这些应用必须预安装在设备上，并且只能在设备的系统软件更新时进行更新。
4. **系统 API（System API）**
    系统 API 表示仅供合作伙伴和 `OEM` 纳入捆绑应用的 Android API。这些 API 在源代码中被标记为 [@Systemapi](https://github.com/Systemapi)。
5. **Android API**
    Android API 是面向第三方 Android 应用开发者的公开 API。
6. **Android 框架（Android Framework）**
    构建应用所依据的一组 Java 类、接口和其他预编译代码。框架的某些部分可通过使用 Android API 公开访问。框架的其他部分只能由 `OEM` 通过系统 API 来访问。Android 框架代码在应用进程内运行。
    
    下面来看这一层所提供的主要组件：

| 名称                          | 功能描述                                  |
| --------------------------- | ------------------------------------- |
| Activity Manager（活动管理器）     | 管理各个应用程序生命周期，以及常用的导航回退功能              |
| Location Manager（位置管理器）     | 提供地理位置及定位功能服务                         |
| Package Manager（包管理器）       | 管理所有安装在 Android 系统中的应用程序              |
| Notification Manager（通知管理器） | 使得应用程序可以在状态栏中显示自定义的提示信息               |
| Resource Manager（资源管理器）     | 提供应用程序使用的各种非代码资源，如本地化字符、图片、布局文件、颜色文件等 |
| Telephony Manager（电话管理器）    | 管理所有的移动设备功能                           |
| Window Manager（窗口管理器）       | 管理所有开启的窗口程序                           |
| Content Provider（内容提供器）     | 使得不同应用程序之间可以共享数据                      |
| View System（视图系统）           | 构建应用程序的基本组件                           |

7. **系统服务（System Services）**
    系统服务是重点突出的模块化组件，例如 `system_server`、`SurfaceFlinger` 和 `MediaService`。Android 框架 API 提供的功能可以与系统服务进行通信，以访问底层硬件。
8. **Android 运行时 (Android Runtime)**
    `AOSP` 提供的 Java 运行时环境。 ART 会将应用的字节码转换为由设备运行时环境执行的处理器专有指令。
9. **硬件抽象层 (HAL)**
    HAL 是一个抽象层，其中包含硬件供应商要实现的标准接口。借助 HAL，Android 可以忽略较低级别的驱动程序实现。借助 HAL，您可以顺利实现相关功能，而不会影响或更改更高级别的系统。
10. **原生守护程序和库（Native Daemons and Libraries）**
    该层中的原生守护程序包括 `init`、`healthd`、`logd` 和 `storaged`。这些守护程序直接与内核或其他接口进行交互，并且不依赖于基于用户空间的 HAL 实现。-
    该层中的原生库包括 `libc`、`liblog`、`libutils`、`libbinder` 和 `libselinux`。这些原生库直接与内核或其他接口进行交互，并且不依赖于基于用户空间的 HAL 实现。
11. **内核（Linux Kernel）**
    内核是任何操作系统的中心部分，并与设备上的底层硬件进行通信。尽可能将 `AOSP` 内核拆分为与硬件无关的模块和特定于供应商的模块。