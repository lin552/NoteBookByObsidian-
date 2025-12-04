---
创建时间: "2025-12-02 17:06:02"
作者: wangxiaoming
tags:
---
#### 一、核心定义与作用
##### 1.定义
`onConfigurationChanged(Configuration newConfig)`是 `ContextThemeWrapper`（Activity 的父类）和 `View`类中的抽象方法，具体实现由子类完成。当系统配置发生变化时，框架会自动调用该方法，并传入包含新配置信息的 `Configuration`对象。

##### 2.核心作用
- **感知配置变化**：让组件知道系统配置（如屏幕方向、语言、字体大小）已改变。
- **动态调整状态**：组件可在不重启的情况下，更新布局、资源、逻辑等以适应新配置（如横竖屏切换时调整控件排列）。
- **避免重建开销**：通过声明 `configChanges`，阻止组件因配置变化而被销毁重建（默认行为），减少性能损耗。

#### 二、触发条件：何时会调用？
`onConfigurationChanged`并非所有配置变化都会触发，需满足两个条件：
##### 1.系统检测到配置变化
Android 系统会监听以下常见配置变化（定义在 `Configuration`类中）：

|**配置类型**​|**说明**​|**示例场景**​|
|---|---|---|
|`orientation`|屏幕方向（横屏/竖屏）|手机旋转|
|`screenSize`|屏幕尺寸（含分屏、折叠屏展开）|分屏模式切换|
|`smallestScreenSize`|最小屏幕尺寸（如平板 vs 手机）|外接显示器|
|`fontScale`|系统字体缩放比例|用户调整系统字体大小|
|`locale`|系统语言/地区|切换中英文|
|`uiMode`|界面模式（白天/黑夜模式）|切换深色模式|
|`keyboardHidden`|键盘可见性（弹出/收起）|外接键盘连接/断开|
##### 2.组件声明`configChanges`属性
默认情况下，若组件（如 Activity）未声明 `configChanges`，系统会在配置变化时**销毁并重建组件**（如 Activity 执行 `onPause`→ `onStop`→ `onDestroy`→ `onCreate`...）。

若希望组件**不重启**，需在 `AndroidManifest.xml`中为组件声明 `configChanges`，指定“无需重启的配置类型”。例如：
```xml
<activity  
    android:name=".MainActivity"  
    android:configChanges="orientation|screenSize|fontScale|locale" />
```
此时，当屏幕旋转（`orientation`）、分屏（`screenSize`）、字体缩放（`fontScale`）或语言切换（`locale`）时，Activity 不会重启，而是直接调用 `onConfigurationChanged`。

#### 三、调用流程：从系统到组件的传递
当配置变化发生且组件声明了 `configChanges`时，调用流程如下：
``` graph TD
    classDef rootView fill:#f9f,stroke:#333;  %% 定义“根View”节点样式（粉色填充）
    classDef childView fill:#9f9,stroke:#333; %% 定义“子View”节点样式（绿色填充）

    A[系统检测配置变化] --> B[ActivityThread 处理配置变更]
    B --> C[调用 Activity.onConfigurationChanged]
    C --> D[Activity 通知根 View（DecorView）]
    D --> E[根 View.dispatchConfigurationChanged]
    E --> F[根 View.onConfigurationChanged]
    class E,F rootView  %% 应用“根View”样式
    E --> G[递归调用所有子 View.dispatchConfigurationChanged]
    G --> H[子 View.onConfigurationChanged]
    class H childView  %% 应用“子View”样式
    G --> I[子 View 的子 View.dispatchConfigurationChanged]
    class I childView
```

- **Activity 层**：系统先调用 Activity 的 `onConfigurationChanged`，开发者可在此处处理全局配置（如更新主题）。
- **View 层**：Activity 通过根 View（`DecorView`）的 `dispatchConfigurationChanged`将事件分发给整个视图树，每个 View 会调用自身的 `onConfigurationChanged`（若重写）。
#### 四、使用场景与代码示例
`onConfigurationChanged`主要用于**动态调整 UI 或资源**，避免组件重启。以下是常见场景：
##### 1）横竖屏切换时调整布局
例如，竖屏时单列布局，横屏时双列布局：
```java
@Override  
public void onConfigurationChanged(Configuration newConfig) {  
    super.onConfigurationChanged(newConfig);  
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {  
        // 横屏：加载双列布局  
        setContentView(R.layout.activity_main_landscape);  
    } else {  
        // 竖屏：加载单列布局  
        setContentView(R.layout.activity_main_portrait);  
    }  
}
```
##### 2）字体缩放时更新UI尺寸
当用户调整系统字体大小（`fontScale`），动态更新控件字号：
```java
@Override  
public void onConfigurationChanged(Configuration newConfig) {  
    super.onConfigurationChanged(newConfig);  
    float fontScale = newConfig.fontScale; // 新字体缩放比例  
    textView.setTextSize(TypedValue.COMPLEX_UNIT_SP, 14 * fontScale); // 按比例调整字号  
}
```
##### 3）语言切换时更新文本资源
当系统语言切换（`locale`），重新加载多语言文本：
```java
@Override  
public void onConfigurationChanged(Configuration newConfig) {  
    super.onConfigurationChanged(newConfig);  
    String currentLang = newConfig.locale.getLanguage(); // 新语言（如 "zh"、"en"）  
    textView.setText(getString(R.string.app_name)); // 重新获取当前语言的资源  
}
```
##### 4）View中响应配置变化
自定义 View 可重写 `onConfigurationChanged`，调整绘制逻辑（如横屏时绘制横向进度条）：
```java
public class CustomView extends View {  
    @Override  
    protected void onConfigurationChanged(Configuration newConfig) {  
        super.onConfigurationChanged(newConfig);  
        if (newConfig.orientation == ORIENTATION_LANDSCAPE) {  
            mIsLandscape = true; // 标记横屏状态  
        } else {  
            mIsLandscape = false;  
        }  
        invalidate(); // 触发重绘  
    }  

    @Override  
    protected void onDraw(Canvas canvas) {  
        if (mIsLandscape) {  
            // 横屏绘制逻辑  
        } else {  
            // 竖屏绘制逻辑  
        }  
    }  
}
```

#### 五、注意事项与避坑指南
`onConfigurationChanged`运行在**主线程**，若执行耗时操作（如同步 IO、复杂计算），会直接阻塞 UI，导致 ANR。以下是关键注意事项：

##### 1）禁止执行耗时操作
- **反面案例**：在 `onConfigurationChanged`中同步解码图片（如 `BitmapFactory.decodeFile`）、解析复杂矢量图（`VectorDrawable.inflate`）、数据库查询等，会导致主线程卡顿（如之前的 670ms 矢量图解析卡顿）。
- **正确做法**：将耗时任务移到后台线程（如用 `AsyncTask`、`RxJava`或线程池），通过 `Handler`回调到主线程更新 UI。
##### 2）避免过度声明configChanges
`configChanges`声明越多，组件需处理的配置变化越复杂，易遗漏逻辑。建议仅声明**必要配置**（如 `orientation|screenSize`），非必要配置（如 `mcc|mnc`运营商变化）可交给系统默认重建处理。

##### 3）View中慎用configurationChanged
View 的 `onConfigurationChanged`通常由父容器的 `dispatchConfigurationChanged`触发（递归分发），若自定义 `ViewGroup`重写 `dispatchConfigurationChanged`并手动调用子 View 的 `onConfigurationChanged`，可能导致**重复调用**。**优先使用 `ViewGroup` 默认分发逻辑**，不重写 `dispatchConfigurationChanged`。

##### 4）正确处理Configuration对象
`newConfig`包含新配置，但部分字段可能为默认值（如未变化的配置）。建议通过 `diff`方法比较新旧配置，仅处理变化的项：
```java
@Override  
public void onConfigurationChanged(Configuration newConfig) {  
    super.onConfigurationChanged(newConfig);  
    int diff = newConfig.diff(mOldConfig); // 比较新旧配置差异  
    if ((diff & ActivityInfo.CONFIG_ORIENTATION) != 0) {  
        // 仅处理方向变化  
    }  
    mOldConfig.setTo(newConfig); // 更新旧配置引用  
}
```

#### 六、与dispatchConfigurationChanged的区别
| **维度**​   | `onConfigurationChanged`    | `dispatchConfigurationChanged`                                             |
| --------- | --------------------------- | -------------------------------------------------------------------------- |
| **所属类**​  | Activity、View、Service 等组件   | ViewGroup（视图容器）                                                            |
| **作用**​   | 组件自身处理配置变化的逻辑               | 将配置变化递归分发给子 View                                                           |
| **调用者**​  | 系统或父容器（如 Activity 通知根 View） | 父容器（如 Activity 或上层 ViewGroup）                                              |
| **核心逻辑**​ | 调整自身状态（布局、资源、逻辑）            | 调用自身 `onConfigurationChanged`+ 递归调用子 View 的 `dispatchConfigurationChanged` |