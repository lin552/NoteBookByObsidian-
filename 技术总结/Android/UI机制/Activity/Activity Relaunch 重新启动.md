---
创建时间: "2025-08-14 18:25:57"
作者: wangxiaoming
tags:
---
在 Android 开发中，​**Activity 的 Relaunch（重新启动）​**​ 是指因系统配置变更（如屏幕旋转）或内存不足导致 Activity 被销毁后重新创建的过程。​**Fragment 作为 Activity 的子组件，其生命周期与 Activity 紧密关联**，但表现更为复杂。以下是详细解析：

#### 一、Activity被销毁阶段
##### 1.Activity被销毁阶段
- ​**触发条件**​：系统配置变更（如屏幕旋转）、内存不足、用户主动重启。
- ​**回调顺序**​：
    `onPause()`→ `onStop()`→ `onDestroy()`
    - ​**数据保存**​：系统自动调用 `onSaveInstanceState(Bundle)`保存临时数据（如用户输入、滚动位置）
    - ​**视图保存**​：系统自动保存 View 层次结构（需为 View 设置 `android:id`）。

##### 2.Activity重建阶段
- **回调顺序**​：
   `onCreate(Bundle savedInstanceState)`→ `onStart()`→ `onResume()`
	- ​**数据恢复**​：通过 `savedInstanceState`恢复数据，重新初始化 `UI`
	- ​**视图恢复**​：系统自动恢复 View 状态（如 `EditText`内容、`RecyclerView`滚动位置）。

#### 二、Fragment在Activity Relaunch中的表现
##### 1.生命周期同步
- **Activity 销毁时**​：Fragment 会依次执行 `onPause()`→ `onStop()`→ `onDestroyView()`→ `onDestroy()`。
- **Activity 重建时**​：Fragment 会重新执行 `onCreate()`→ `onCreateView()`→ `onViewCreated()`→ `onActivityCreated()`→ `onStart()`→ `onResume()`
##### 2.状态保存与恢复
- 自动保存：
	Fragment的`onSaveInstanceState(Bundle)`会被调用，保存临时数据（如网络请求状态）。
- 手动恢复：
	在Fragment的`onCreateView()`或`onActivityCreated()`中，通过`saveInstanceState`恢复数据
##### 3.常见问题与解决方案
| 问题现象               | 原因分析                                                  | 解决方案                                                                  |
| ------------------ | ----------------------------------------------------- | --------------------------------------------------------------------- |
| ​**Fragment 重叠**​  | Activity 重建时，`FragmentManager`默认恢复所有 Fragment，未隐藏旧实例。 | 使用 `findFragmentByTag()`获取已存在的 Fragment，并通过 `hide()`控制显示              |
| ​**Fragment 空指针**​ | 异步任务完成后调用 `getActivity()`，但 Fragment 已分离。             | 在 Fragment 中缓存 `Activity`引用（如 `mActivity`），避免直接调用 `getActivity()`<br> |
| ​**界面空白**​         | Fragment 未正确恢复状态，或视图未重新绑定。                            | 在 `onCreateView()`中检查 `savedInstanceState`，重新初始化视图逻辑<br>              |
#### 三、关键代码示例
##### 1.Activity 中保存Fragment状态
```java
// MainActivity.java
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    // 保存当前显示的 Fragment 标签
    outState.putString("currentFragment", "HomeFragment");
}

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    if (savedInstanceState != null) {
        String currentFragmentTag = savedInstanceState.getString("currentFragment");
        // 恢复 Fragment
        getSupportFragmentManager().beginTransaction()
            .show(getSupportFragmentManager().findFragmentByTag(currentFragmentTag))
            .commit();
    } else {
        // 首次加载默认 Fragment
        getSupportFragmentManager().beginTransaction()
            .add(R.id.container, new HomeFragment(), "HomeFragment")
            .commit();
    }
}
```

##### 2.Fragment中恢复数据
```java
// HomeFragment.java
private String mData;

@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if (savedInstanceState != null) {
        mData = savedInstanceState.getString("data");
    }
}

@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_home, container, false);
    TextView textView = view.findViewById(R.id.textView);
    textView.setText(mData != null ? mData : "Default Text");
    return view;
}

@Override
public void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putString("data", "Saved Content");
}
```

#### 四、优化策略
1. ​**避免内存泄漏**​
    - 在 Fragment 中缓存 `Activity`引用时，需在 `onDestroyView()`中置空
2. ​**处理异步任务**​
    - 使用 `ViewModel`或 `LiveData`管理数据，避免因 Activity 重建导致异步回调失效。
3. ​**配置 `setRetainInstance(true)`**​
    - 保留 Fragment 实例（适用于需跨配置变更保持状态的场景）

#### 五、调试与验证
##### 1.`ADB`命令
```bash
adb shell dumpsys activity fragments  # 查看 Fragment 状态
adb logcat | grep "Fragment"         # 监控 Fragment 生命周期
```
##### 2.日志标记
在 Fragment 生命周期方法中添加日志，观察重建流程
```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    Log.d("FragmentLifecycle", "onCreate: " + (savedInstanceState != null));
    super.onCreate(savedInstanceState);
}
```