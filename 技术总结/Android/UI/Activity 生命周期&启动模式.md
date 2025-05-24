---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Activity
---
#### 一、Activity生命周期
![[Pasted image 20250323202548.png]]
##### 1）核心生命周期方法
- ​**`onCreate()`**：首次创建时调用，用于初始化布局、绑定数据等基础工作（如调用`setContentView()`）
- ​**`onStart()`**：Activity可见但不可交互，适合注册广播或恢复资源
- ​**`onResume()`**：进入前台并获取焦点，可恢复动画、传感器监听等操作
- ​**`onPause()`**：失去焦点（如弹窗覆盖），需释放资源（如相机、传感器），但避免耗时操作
- ​**`onStop()`**：完全不可见时调用，可保存数据（如草稿）或停止后台任务
- ​**`onDestroy()`**：最终销毁前调用，释放所有残留资源
- ​**`onRestart()`**：从`onStop()`返回到前台时触发，通常与`onStart()`和`onResume()`联动

##### 2) 典型状态转换场景
- ​**正常启动到退出**：`onCreate → onStart → onResume → onPause → onStop → onDestroy`
- ​**切换至其他应用/点击HOME键**：`onPause → onStop`，返回时触发`onRestart → onStart → onResume`
- ​**横竖屏切换/配置变化**：Activity销毁并重建，触发`onSaveInstanceState`保存临时数据，重建后通过`onRestoreInstanceState`恢复
```kotlin
lateinit var textView: TextView
var gameState: String? = null

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    gameState = savedInstanceState?.getString(GAME_STATE_KEY)
    setContentView(R.layout.activity_main)
    textView = findViewById(R.id.text_view)
}

override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
}

override fun onSaveInstanceState(outState: Bundle?) {
    outState?.run {
        putString(GAME_STATE_KEY, gameState)
        putString(TEXT_VIEW_KEY, textView.text.toString())
    }
    super.onSaveInstanceState(outState)
}
```

- **A -> B 页面 A 和 B 的生命周期顺序** `A:onPause - > B:onCreate -> onStart() -> onResume() ->A:onStop()`  如果B是透明主题或是一个DialogActivity,则不会调 `A:onSotp()`

##### 3)Activity 启动过程

![[Pasted image 20250323205100.png]]

```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    ...
    ActivityInfo aInfo = r.activityInfo;
    if (r.packageInfo == null) {
        //step 1: 创建LoadedApk对象
        r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                Context.CONTEXT_INCLUDE_CODE);
    }
    ... //component初始化过程

    java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
    //step 2: 创建Activity对象
    Activity activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    ...

    //step 3: 创建Application对象
    Application app = r.packageInfo.makeApplication(false, mInstrumentation);

    if (activity != null) {
        //step 4: 创建ContextImpl对象
        Context appContext = createBaseContextForActivity(r, activity);
        CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
        Configuration config = new Configuration(mCompatConfiguration);
        //step5: 将Application/ContextImpl都attach到Activity对象
        activity.attach(appContext, this, getInstrumentation(), r.token,
                r.ident, app, r.intent, r.activityInfo, title, r.parent,
                r.embeddedID, r.lastNonConfigurationInstances, config,
                r.referrer, r.voiceInteractor);

        ...
        int theme = r.activityInfo.getThemeResource();
        if (theme != 0) {
            activity.setTheme(theme);
        }

        activity.mCalled = false;
        if (r.isPersistable()) {
            //step 6: 执行回调onCreate
            mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
        } else {
            mInstrumentation.callActivityOnCreate(activity, r.state);
        }

        r.activity = activity;
        r.stopped = true;
        if (!r.activity.mFinished) {
            activity.performStart(); //执行回调onStart
            r.stopped = false;
        }
        if (!r.activity.mFinished) {
            //执行回调onRestoreInstanceState
            if (r.isPersistable()) {
                if (r.state != null || r.persistentState != null) {
                    mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                            r.persistentState);
                }
            } else if (r.state != null) {
                mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
            }
        }
        ...
        r.paused = true;
        mActivities.put(r.token, r);
    }

    return activity;
}

```

##### 4)使用注意事项
- 在`onPause()`中释放资源，在`onStop()`中保存数据，避免在`onDestroy()`中处理关键逻辑
- 使用`ViewModel`+`onSaveInstanceState`分离临时数据与持久化数据

#### 二、Activity启动模式
##### 1)`standard`(默认)
特性：每次启动均创建新实例，栈中允许多个相同Activity
适用场景：适用于独立页面（如新闻详情页）

##### 2)`singleTop`(栈顶复用)
特性：若目标Activity已在栈顶，则复用实例并调用 `onNewIntent()`,否则创建新实例
适用场景：适用于避免重复打开同意页面（如消息通知页）

##### 3）`singleTask` (栈内复用)
特性：保证整个任务栈中仅存在一个实例，若已存在，则清楚其上的所有Activity并调用 `onNewIntent()`.
适用场景：适用于应用入口 (如主页)

##### 4）`singleInstance` （全局唯一）
特性：为Activity分配独立任务栈，且栈内仅此实例。
适用场景：适用于需独立运行的场景（如分享弹窗、视频通话页面）