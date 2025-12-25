---
创建时间: "2025-12-24 14:25:58"
作者: wangxiaoming
tags:
---
Fragment 回退栈本质上是一个**事务栈**。每当你执行一个会改变 Fragment 状态的事务（如添加、替换、移除）并调用 `addToBackStack(String name)`时，这个事务就会被压入栈中。
- **用户按返回键**：系统会从栈顶弹出最近的一个事务，并执行其**反向操作**，从而恢复到之前的 Fragment 状态。
- **目的**：它模拟了 Activity 的任务栈行为，让用户在 Fragment 之间导航时有清晰的“后退”路径，而不是简单地退出应用。

#### 一、入栈和出栈
##### 1）入栈-记录状态变化
在你提交一个事务时，通过调用 `addToBackStack(String name)`将其加入回退栈。
name： 是一个可选的字符串标识符，用于在之后精确控制回退栈。如果传 null，则无法按名称回退。
```java
// 开始一个事务
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
// 替换容器中的 Fragment
transaction.replace(R.id.fragment_container, NewFragment.class, null);
// ！！！关键步骤：将此事务加入回退栈
transaction.addToBackStack("replacing_with_new_fragment"); 
// 提交事务
transaction.commit();
```
在这个例子中：

- 用户从 `FragmentA`导航到 `FragmentB`。
- 事务 `(FragmentA -> FragmentB)`被压入栈。
- 此时用户按返回键，栈顶事务被弹出，执行反向操作 `(FragmentB -> FragmentA)`，用户回到了 `FragmentA`。
##### 2）出栈-导航回退
主要有两种方式触发出栈
- **用户按下系统返回键**：这是最常见的方式。系统会自动从回退栈中弹出顶部事务。
- **代码中主动调用**：你可以编程式地控制回退栈。
```java
// 方式1：弹出栈顶的一个事务（相当于按一次返回键）
getSupportFragmentManager().popBackStack();

// 方式2：弹出栈顶，并立即同步执行
getSupportFragmentManager().popBackStackImmediate();

// 方式3：弹出直到指定的事务名称（不包含该名称的事务）
// 例如，栈里有 [T1, T2, T3]，pop到 T1，则 T2, T3 被弹出，留下 T1
getSupportFragmentManager().popBackStack("T1", 0);

// 方式4：弹出直到指定的事务名称（包含该名称的事务）
// 例如，栈里有 [T1, T2, T3]，pop到 T1，则 T1, T2, T3 全部被弹出，栈为空
getSupportFragmentManager().popBackStack("T1", FragmentManager.POP_BACK_STACK_INCLUSIVE);
```

#### 二、高级控制与管理
##### 1）获取回退栈信息
`getBackStackEntryCount()`: 获取回退栈中事务的数量。
`getBackStackEntryAt(int index)`: 根据索引获取一个 BackStackEntry对象，它可以提供该事务的名称、ID 等信息
```java
int count = getSupportFragmentManager().getBackStackEntryCount();
if (count > 0) {
    FragmentManager.BackStackEntry entry = getSupportFragmentManager().getBackStackEntryAt(count - 1);
    String lastTransactionName = entry.getName(); // 获取最后一个事务的名称
}
```
##### 2)监听回退栈变化
你可以添加一个 `OnBackStackChangedListener`来监听回退栈的任何变化。
```java
getSupportFragmentManager().addOnBackStackChangedListener(new FragmentManager.OnBackStackChangedListener() {
    @Override
    public void onBackStackChanged() {
        // 当回退栈发生变化时（入栈、出栈、清空等），这里会被回调
        int nowCount = getSupportFragmentManager().getBackStackEntryCount();
        // 可以在这里更新 UI，比如改变 ActionBar 的 Up 按钮显示逻辑
    }
});
```
**注意**：记得在适当的时候（如 `onDestroyView`）移除监听器，防止内存泄漏。
