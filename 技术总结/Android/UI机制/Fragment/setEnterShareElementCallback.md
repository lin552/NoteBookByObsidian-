---
创建时间: "2025-12-01 15:24:33"
作者: wangxiaoming
tags:
---
`setEnterSharedElementCallback`是 Android 中用于**控制 Activity/Fragment 间共享元素进入过渡（Shared Element Transition）**的核心方法，通过设置回调监听共享元素进入的全过程，允许开发者自定义过渡行为、处理元素映射、调整动画状态等。

#### 一、基础概念：共享元素过渡（Shared Element Transition）
共享元素过渡是指**两个界面（如 Activity A → Activity B）中，相同语义的视图（如图片、卡片）在切换时平滑移动的动画**（例如：A 界面的缩略图放大过渡到 B 界面的全屏图）。这种过渡依赖“共享元素”的身份映射（A 中的元素与 B 中的元素对应），而 `setEnterSharedElementCallback`正是用于管理**目标界面（B）中共享元素“进入”时的逻辑**。

#### 二、方法定义与作用范围
`setEnterSharedElementCallback`主要有两种使用场景：
##### 1)Activity中使用
通过 `Activity`的 `Window`对象设置，作用于**新启动的 Activity（目标 Activity）**​ 的共享元素进入过渡：
```java
// 在目标 Activity（如 B）的 onCreate 中设置
getWindow().setEnterSharedElementCallback(new SharedElementCallback() {
    // 重写回调方法...
});
```
##### 2)Fragment中使用
通过 `FragmentTransaction`或 `Fragment`自身设置（需 API 21+），作用于**被添加的 Fragment（目标 Fragment）**​ 的共享元素进入过渡：
```java
// 在 FragmentTransaction 中添加共享元素时设置
fragmentTransaction.addSharedElement(sharedElement, "shared_element_name")
                  .setReorderingAllowed(true) // 必须开启重排序以支持共享元素
                  .setEnterSharedElementCallback(new SharedElementCallback() {
                      // 重写回调方法...
                  })
                  .commit();
```
#### 三、核心作用：通过SharedElementCallback回调控制过渡
`setEnterSharedElementCallback`的参数是一个 `SharedElementCallback`实例（抽象类），其内部定义了多个关键回调方法，用于干预共享元素进入的各个环节。**最核心的作用是“监控和调整共享元素进入过渡的细节”**，具体通过以下方法实现：
##### 1) `onMapSharedElements(List<String> names, Map<String, View> sharedElements)`

- **作用**：**映射共享元素名称与实际视图**。
    源界面（A）会通过共享元素名称（如 `"image"`）标识要过渡的元素，目标界面（B）需通过该回调将这些名称映射到自身的实际视图（如 B 中的 `ImageView`）。
- **典型场景**：
    - 动态修改共享元素（如根据数据替换图片）；
    - 修复名称不匹配问题（如源界面名称与 B 中视图 ID 不一致时，手动映射）；
    - 过滤不需要的共享元素（从 `names`列表中移除无效名称，或从 `sharedElements`中移除对应视图）。
```java
@Override
public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
    // 示例：将源界面的 "user_avatar" 映射到 B 中的实际 ImageView
    if (names.contains("user_avatar")) {
        View targetView = findViewById(R.id.iv_avatar_b); // B 中的目标视图
        sharedElements.put("user_avatar", targetView);
    }
    // 移除无效名称（避免过渡失败）
    names.remove("invalid_name");
}
```
##### 2) `onSharedElementStart(List<String> names, List<View> sharedElements, List<View> sharedElementSnapshots)`


- **作用**：**共享元素进入过渡开始时回调**。
    此时源界面（A）的共享元素已开始隐藏，目标界面（B）的共享元素即将显示，可获取初始状态（如位置、尺寸）并执行预处理（如临时调整视图属性）。
- **参数说明**：
    - `names`：共享元素名称列表；
    - `sharedElements`：目标界面（B）中映射后的共享元素视图列表；
    - `sharedElementSnapshots`：源界面（A）共享元素的快照视图列表（可用于对比或叠加效果）。
##### 3）`onSharedElementEnd(List<String> names, List<View> sharedElements, List<View> sharedElementSnapshots)`

**作用**：**共享元素进入过渡结束时回调**。
过渡动画完成，共享元素已到达最终位置，可执行收尾操作（如恢复视图临时修改的属性、清理快照资源）。

##### 4） `onRejectSharedElements(List<View> rejectedSharedElements)`

**作用**：**处理被拒绝的共享元素**。
当源界面（A）的共享元素无法在目标界面（B）中找到对应映射时，这些元素会被放入 `rejectedSharedElements`列表，可在此处手动处理（如移除、替换为默认视图）。

##### 5）`onCaptureSharedElementSnapshot(View sharedElement, Matrix viewToGlobalMatrix, RectF screenBounds)`

**作用**：**捕获共享元素的快照**（用于过渡动画的视觉衔接）。
系统默认会生成快照，但可重写此方法自定义快照（如添加滤镜、裁剪）。

#### 四、典型使用场景
1. **修复共享元素映射失败**​
    当源界面与目标界面的共享元素名称或视图 ID 不匹配时，通过 `onMapSharedElements`手动映射名称与视图。
2. **动态调整共享元素状态**​
    在 `onSharedElementStart`中临时修改共享元素（如隐藏文字、调整透明度），过渡结束后再恢复（`onSharedElementEnd`）。
3. **处理复杂过渡逻辑**​
    例如：共享元素进入时，伴随额外的缩放、旋转动画，可通过回调获取元素初始/最终位置，结合属性动画实现。
4. **兼容低版本或特殊机型**​
    部分机型对共享元素过渡支持不完善，可通过回调降级处理（如禁用过渡、改用普通动画）。
#### 五、注意事项
- **与 `setExitSharedElementCallback`的区别**：
    `setEnterSharedElementCallback`作用于**目标界面（进入方）**，而 `setExitSharedElementCallback`作用于**源界面（退出方）**，分别控制进入和退出的过渡。
    
- **必须在过渡开始前设置**：
    回调需在 `startActivity`或 `FragmentTransaction.commit()`前设置，否则可能不生效。
    
- **避免过度干预**：
    系统默认的共享元素过渡已优化性能，仅在必要时重写回调（如修复映射问题、添加自定义效果）。