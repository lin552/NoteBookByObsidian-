---
创建时间: "2025-12-01 15:35:21"
作者: wangxiaoming
tags:
---
`setExitSharedElementCallback`是 Android 共享元素过渡（Shared Element Transition）中，用于控制**源界面（退出方）共享元素“退出”过程**的核心回调 API。它与 `setEnterSharedElementCallback`（控制目标界面“进入”过程）对称，共同构成共享元素过渡的完整控制体系。

#### 一、核心定位：源界面的“退出管家”
当从一个界面（**源界面 A**，如列表页）跳转到另一个界面（**目标界面 B**，如详情页）时，共享元素（如列表项中的图片）会从 A 移动到 B。此时：
- `setExitSharedElementCallback`作用于**源界面 A**，控制 A 中共享元素“离开”时的动画、状态调整和引用管理；
- `setEnterSharedElementCallback`作用于**目标界面 B**，控制 B 中共享元素“进入”时的逻辑。

#### 二、适用场景与设置方式
##### 1. **Activity 中使用**​
在**源 Activity A**​ 中，通过 `Window`对象设置，作用于 A 中共享元素的退出过渡：
```java
// 在源 Activity A 的 onCreate 或 startActivity 前设置
getWindow().setExitSharedElementCallback(new SharedElementCallback() {
    // 重写退出过渡的回调方法...
});
```
##### 2.Fragment中使用
在**源 Fragment A**​ 中，通过 `FragmentTransaction`或 `Fragment`自身设置（API 21+），作用于 A 中共享元素的退出过渡：
```java
// 在源 Fragment A 的 commit 事务前设置（通过 FragmentTransaction）
FragmentTransaction transaction = getParentFragmentManager().beginTransaction();
transaction.addSharedElement(sharedElementInA, "shared_element_name") // 声明共享元素
           .setExitSharedElementCallback(new SharedElementCallback() { // 设置退出回调
               // 重写回调方法...
           })
           .replace(R.id.container, TargetFragmentB.class, args)
           .commit();
```
#### 三、核心回调方法：控制退出过度的细节
`setExitSharedElementCallback`的参数是 `SharedElementCallback`抽象类实例，需重写其方法干预退出过程。以下是**退出场景特有的关键作用**（与进入回调方法名相同，但作用于源界面）：
##### 1) `onMapSharedElements(List<String> names, Map<String, View> sharedElements)`

- **作用**：**映射源界面共享元素名称与实际视图**（修正或过滤退出元素）。
    源界面 A 中，系统会根据 `names`列表（来自目标界面 B 的请求）查找对应视图；若名称与视图 ID 不匹配（如动态布局导致 ID 变化），可在此手动映射。
- **典型场景**：
    - 过滤无需退出的共享元素（从 `names`移除无效名称，或从 `sharedElements`移除对应视图）；
    - 动态替换退出视图（如根据数据更新图片后再退出）。
```java
@Override
public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
    // 示例1：将源界面的 "item_image" 映射到实际视图（若名称与 ID 不一致）
    if (names.contains("item_image")) {
        View sourceView = findViewById(R.id.iv_list_item); // 源界面 A 中的实际视图
        sharedElements.put("item_image", sourceView);
    }
    // 示例2：移除无效名称（避免退出过渡失败）
    names.remove("obsolete_name");
}
```

##### 2) `onSharedElementStart(List<String> names, List<View> sharedElements, List<View> sharedElementSnapshots)`

- **作用**：**共享元素退出过渡开始时回调**
    此时源界面 A 的共享元素即将隐藏，目标界面 B 的共享元素开始进入。可获取源元素初始状态（位置、尺寸），执行预处理（如临时放大、修改透明度）。
- **参数说明**：
    - `names`：共享元素名称列表（来自目标界面 B 的请求）；
    - `sharedElements`：源界面 A 中映射后的共享元素视图列表；
    - `sharedElementSnapshots`：源界面 A 共享元素的快照（可用于对比或叠加效果）。

##### 3) `onSharedElementEnd(List<String> names, List<View> sharedElements, List<View> sharedElementSnapshots)`

**作用**：**共享元素退出过渡结束时回调**。
退出动画完成，源界面 A 的共享元素已隐藏。可执行收尾操作：
- 恢复视图临时修改的属性（如退出前放大的元素恢复原尺寸）；
- 清理快照资源（`sharedElementSnapshots`需手动释放避免内存泄漏）；
- 移除源界面中为过渡添加的临时监听器（如动画监听）。

##### 4) `onRejectSharedElements(List<View> rejectedSharedElements)`

**作用**：**处理被拒绝的共享元素**。
当目标界面 B 请求的共享元素名称在源界面 A 中找不到对应视图（如 `onMapSharedElements`未正确映射），这些元素会被放入 `rejectedSharedElements`列表。可在此手动处理（如移除、替换为默认视图），避免过渡崩溃。

##### 5) `onCaptureSharedElementSnapshot(View sharedElement, Matrix viewToGlobalMatrix, RectF screenBounds)`

**作用**：**捕获源界面共享元素的快照**（用于退出过渡的视觉衔接）。
系统默认生成快照，但可重写此方法自定义（如添加边框、模糊效果），确保退出动画与进入动画的视觉连贯性。

#### 四、典型使用场景
##### 1. **修复源界面共享元素映射失败**​
当源界面 A 的视图 ID 与目标界面 B 请求的名称不匹配时（如列表项复用导致 ID 动态变化），通过 `onMapSharedElements`手动映射名称与视图，避免退出过渡卡顿或失败。
##### 2. **动态调整源界面共享元素状态**​
在 `onSharedElementStart`中临时修改源元素（如退出前将图片放大 10%，增强过渡动感），在 `onSharedElementEnd`中恢复原状态。
##### 3. **清理退出过渡的临时资源**​
在 `onSharedElementEnd`中释放快照资源（`sharedElementSnapshots`）、移除动画监听器，避免内存泄漏（尤其列表页频繁跳转时）。
##### 4. **处理复杂退出逻辑**​
例如：源界面 A 的共享元素退出时，伴随“缩小+淡出”动画，可通过 `onSharedElementStart`获取初始位置，结合 `ObjectAnimator`自定义动画。

#### 五、与`setEnterSharedElementCallback`的核心区别
|**维度**​|`setExitSharedElementCallback`|`setEnterSharedElementCallback`|
|---|---|---|
|**作用对象**​|源界面（退出方 A）|目标界面（进入方 B）|
|**核心目标**​|控制 A 中共享元素“离开”的动画与状态|控制 B 中共享元素“进入”的动画与状态|
|**典型回调关注点**​|映射源视图、清理临时资源、调整退出状态|映射目标视图、初始化进入状态、处理快照|
#### 六、注意事项
1. **必须在跳转前设置**：回调需在 `startActivity`（Activity）或 `FragmentTransaction.commit()`（Fragment）前设置，否则不生效。
2. **避免过度干预**：系统默认退出过渡已优化性能，仅在必要时重写（如修复映射、添加自定义效果）。
3. **弱引用管理**：若在回调中持有源界面视图/对象的强引用，需在 `onSharedElementEnd`中主动置空，避免源界面销毁后泄漏。