---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Android
  - Fragment
---
#### 一、Fragment生命周期
![[Pasted image 20250324100501.png]]
#### 二、Activity管理Fragment的核心方式
##### 1)`FragmentManager` 与 `FragmentTransaction`
- ​`FragmentManager`：通过 `getSupportFragmentManager()` 获取，负责 Fragment 的 ​**事务管理** 和 ​**状态跟踪**
- ​`FragmentTransaction`：用于执行具体操作：
```java
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
transaction.add(R.id.container, new MyFragment(), "TAG"); // 添加
transaction.replace(R.id.container, new AnotherFragment()); // 替换
transaction.remove(oldFragment); // 移除
transaction.addToBackStack(null); // 加入返回栈
transaction.commit();
```
**添加方式**：
- ​**静态添加**：XML 中 `<fragment>` 标签声明（适用于固定布局）
- ​**动态添加**：运行时通过 `add()`/`replace()` 灵活控制

##### 2)返回栈管理
通过 `addToBackStack()` 将事务加入返回栈，用户按返回键时可逐级回退 Fragment 状态
```java
transaction.replace(R.id.container, new DetailFragment());
transaction.addToBackStack("detail_page"); // 命名事务
transaction.commit();
```
##### 3)Fragment状态恢复
- ​`Bundle` 保存：在 `onSaveInstanceState()` 中保存关键数据，通过 `getArguments()` 在 `onCreate()` 中恢复
- ​`ViewModel`：跨配置变更（如屏幕旋转）保留数据，实现 Fragment 间共享状态

#### 三、通信机制 Activity 与 Fragment 交互
##### 1）接口回调
Fragment定义接口，由Activity实现
```java
// Fragment 中
public interface OnDataListener {
    void onDataReceived(String data);
}
private OnDataListener mListener;

@Override
public void onAttach(Context context) {
    super.onAttach(context);
    mListener = (OnDataListener) context; // Activity 需实现接口
}
```
Activity 通过回调方法接收数据
##### 2）Fragment Result API
使用 `setFragmentResult()` 和 `setFragmentResultListener()` 传递一次性数据，避免直接耦合
```kotlin
// 发送方 Fragment
setFragmentResult("requestKey", bundleOf("data" to "value"))

// 接收方 Fragment/Activity
childFragmentManager.setFragmentResultListener("requestKey") { key, bundle ->
    val data = bundle.getString("data")
}
```
##### 3)`ViewModel`共享数据
通过 `ViewModelProvider(activity).get(SharedViewModel::class.java)` 实现跨 Fragment 数据同步

