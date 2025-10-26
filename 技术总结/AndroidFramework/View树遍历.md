---
创建时间: "2025-06-24 15:26:49"
作者: wangxiaoming
tags:
---
在 Android 中，`ViewGroup` 遍历子 `View` 的顺序**既不是深度优先（`DFS`）也不是广度优先（`BFS`）​**，而是基于子 `View` 在内部列表中的**线性顺序（添加顺序）​**。以下是详细分析：

#### 一、子View的存储结构
`ViewGroup` 内部通过一个 `ArrayList<View>` 类型的成员变量 `mChildren` 存储子 `View`（源码可见 `ViewGroup.java` 中的 `mChildren` 数组）。子 `View` 按照添加顺序（通过 `addView(View child)` 方法）依次存入 `mChildren` 列表，索引从 `0` 开始递增。

#### 二、常规遍历顺序：线性顺序（添加顺序）
`ViewGroup` 遍历子 `View` 的默认逻辑是**按 `mChildren` 列表的索引顺序逐个访问**，即从第一个添加的子 `View`（索引 `0`）到最后一个添加的子 `View`（索引 `mChildrenCount-1`）。这一规则适用于大多数场景，包括：
##### 1. 事件分发（`dispatchTouchEvent`）
当 `ViewGroup` 不拦截事件（`onInterceptTouchEvent` 返回 `false`）时，会遍历 `mChildren` 列表，寻找**第一个能接收当前触摸事件的子 `View`**。具体逻辑如下：
```java
// ViewGroup.dispatchTouchEvent() 源码片段（简化）
for (int i = 0; i < mChildrenCount; i++) {
    final View child = getChildAt(i);
    if (!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)) {
        continue; // 跳过不可见或触摸点不在其范围内的子View
    }
    // 找到第一个符合条件的子View，将事件传递给它
    handled = child.dispatchTouchEvent(event);
    break; // 仅处理第一个匹配的子View
}
```
遍历顺序严格遵循 `mChildren` 列表的索引顺序（`i=0` → `i=1` → ...）。

##### 2. 测量（`measure`）与布局（`layout`）
在 `measureChildWithMargins` 或 `layout` 方法中，`ViewGroup` 同样按 `mChildren` 的顺序遍历子 `View`，依次调用子 `View` 的 `measure()` 或 `layout()` 方法：
```java
// ViewGroup.layout() 源码片段（简化）
for (int i = 0; i < mChildrenCount; i++) {
    View child = getChildAt(i);
    child.layout(l, t, r, b); // 按顺序布局每个子View
}
```

#### 三、特殊场景：逆序遍历的例外
虽然默认是线性顺序，但在**事件分发**中，当触摸事件未被任何子 `View` 消费时，`ViewGroup` 会**逆序遍历子 `View`**​ 以寻找可能的“意外”处理者（例如，父 `ViewGroup` 自身需要处理未被子 `View` 消费的 `ACTION_UP` 事件）。例如：
```java
// ViewGroup.dispatchTouchEvent() 源码（处理ACTION_UP时的逆序检查）
if (actionMasked == MotionEvent.ACTION_UP) {
    for (int i = mChildrenCount - 1; i >= 0; i--) { // 逆序遍历
        final View child = getChildAt(i);
        if (child.canReceivePointerEvents() && isTransformedTouchPointInView(x, y, child, null)) {
            child.dispatchTouchEvent(event); // 最后一个可能接触事件的子View
            break;
        }
    }
}
```
这种逆序遍历是为了兼容某些边缘场景（如手指抬起时可能落在子 `View` 边界外），但属于特殊处理，并非常规遍历逻辑。

#### 四、关键结论
|场景|遍历顺序|依据|
|---|---|---|
|常规事件分发（`ACTION_DOWN`/`MOVE`）|线性顺序（添加顺序，`i=0→n-1`）|`ViewGroup.dispatchTouchEvent()` 中按 `mChildren` 索引顺序遍历|
|未被消费的事件（如`ACTION_UP`）|逆序顺序（`i=n-1→0`）|特殊场景下的兼容逻辑，确保边界情况的事件处理|
|测量（`measure`）与布局（`layout`）|线性顺序（添加顺序）|`measureChildWithMargins` 和 `layout()` 方法中按 `mChildren` 顺序调用|
#### 五、总结
`ViewGroup` 遍历子 `View` 的核心规则是**基于 `mChildren` 列表的线性顺序**​（添加顺序），仅在极少数特殊事件场景（如 `ACTION_UP`）下会逆序遍历。理解这一规则有助于定位事件冲突（如滑动冲突）或自定义 `ViewGroup` 时的布局/事件逻辑设计。
