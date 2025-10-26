---
创建时间: 2025-06-24 14:46:20
作者: wangxiaoming
tags:
---
#### 一、动画分类与核心原理

##### 1. ​**动画类型**​
- ​**补间动画（Tween Animation）​**​
    - ​**类型**​：Alpha（透明度）、Scale（缩放）、Translate（位移）、Rotate（旋转）。
    - ​**原理**​：通过改变 View 的绘制效果（矩阵变换），​**不改变实际属性**​（如点击区域不变）
    - ​**XML 实现**​：在 `res/anim` 目录下定义，如 `alpha.xml`。
    - ​**缺点**​：动画结束后 View 回复原位（需设置 `fillAfter=true` 保持状态）

- ​**属性动画（Property Animation）​**​
    - ​**核心类**​：`ValueAnimator`（数值变化）、`ObjectAnimator`（直接操作对象属性）、`AnimatorSet`（组合动画）。
    - ​**原理**​：动态修改对象属性值（如 `translationX`），​**真实改变属性**，支持任意对象
    - ​**插值器（`Interpolator`）​**​：控制动画速度曲线（如 `AccelerateDecelerateInterpolator` 先加速后减速）。
    - ​**估值器（`TypeEvaluator`）​**​：计算属性中间值（如 `FloatEvaluator` 处理浮点变化）

- ​**帧动画（Frame Animation）​**​
    - ​**原理**​：通过顺序播放一系列图片（`animation-list` XML 定义），模拟连续动作
    - ​**缺点**​：资源占用大，易导致内存问题。
    
- ​**转场动画（Transition Animation）​**​
    - ​**场景**​：Activity/Fragment 切换动画（如共享元素过渡）。
    - ​**实现**​：`TransitionManager` + `Scene`，或 XML 定义 `transition.xml`

##### 2.核心原理
- ​**矩阵变换**​：所有动画底层依赖 `Matrix` 类实现坐标变换（平移、缩放、旋转）
- ​**Choreographer**​：协调动画帧刷新，与 `VSYNC` 信号同步，保证流畅性（`60FPS`）
- ​**属性动画流程**​：
    1. 设置起始值和结束值。
    2. 插值器计算当前进度。
    3. 估值器生成当前属性值。
    4. 更新 View 属性并触发重绘

#### 二、高频面试题与实战问题
##### 1. ​**补间动画 vs 属性动画**​
- ​**区别**​：
    - 补间动画仅改变绘制效果，属性动画修改实际属性（如 `translationX`）。
    - 属性动画支持复杂组合（如同时缩放和旋转），补间动画需用 `AnimationSet`
- ​**适用场景**​：
    - 补间动画：简单交互反馈（如按钮点击缩放）。
    - 属性动画：复杂状态变化（如列表项滑动动画）
##### 2. ​**属性动画内存泄漏**​
- ​**原因**​：未取消未完成的动画（如 `Animator` 未调用 `cancel()`），导致 Activity 无法回收。
- ​**解决**​：在 `onDestroy()` 中取消动画，或使用 `WeakReference` 持有动画对象

##### 3. ​**插值器与估值器**​
- ​**自定义插值器**​：继承 `TimeInterpolator`，重写 `getInterpolation(float input)`。
```java
public class BounceInterpolator implements TimeInterpolator {
    @Override
    public float getInterpolation(float input) {
        // 实现弹跳效果
        return (float) (Math.pow(2, -10 * input) * Math.sin((input - 0.1) * (2 * Math.PI) / 0.4) + 1);
    }
}
```
**自定义估值器**​：继承 `TypeEvaluator<T>`，重写 `evaluate(float fraction, T startValue, T endValue)`
##### 4. ​**属性动画的局限性**​
- ​**不支持 View 的属性回调**​：如 `onAnimationEnd` 需手动设置监听器。
- ​**复杂路径动画**​：需结合 `PathInterpolator` 或第三方库（如 Lottie）
##### 5. ​**性能优化**​
- ​**硬件加速**​：开启 `setLayerType(LAYER_TYPE_HARDWARE, null)`，利用 GPU 渲染。
- ​**减少过度绘制**​：避免多层动画叠加，使用 `Hierarchy Viewer` 分析视图层级。
- ​**帧率控制**​：复杂动画使用 `FrameMetrics` 监控帧率，避免掉帧

#### 三、高级动画技术
#####  ​1.Lottie 动画​
- ​**原理**​：解析 Adobe After Effects 导出的 `JSON` 文件，实时渲染矢量动画。
- ​**优点**​：体积小、跨平台、支持复杂效果（如粒子动画）。
```groovy
implementation 'com.airbnb.android:lottie:5.2.0'
```

```xml
<com.airbnb.lottie.LottieAnimationView
    android:id="@+id/animation_view"
    app:lottie_fileName="animation.json"/>
```

##### 2.矢量动画（`AnimatedVectorDrawable`）
**实现路径动画**​：通过 XML 定义路径变形。
```xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_vector">
    <target
        android:name="path"
        android:animation="@anim/path_anim"/>
</animated-vector>
```

##### 3.共享元素转场
**实现**​：在 `ActivityOptions` 中指定共享元素。
```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(
    this, view, "shared_element");
startActivity(intent, options.toBundle());
```

#### 四、源码与设计模式
##### 1. ​**动画状态机**​
- ​**Animator 类**​：通过状态机管理动画生命周期（`STARTED`、`RUNNING`、`END`）。
- ​**事件分发**​：`AnimatorUpdateListener` 监听每一帧更新。
##### 2. ​**观察者模式**​
- ​`AnimatorListener`​：监听动画事件（开始、结束、取消、重复）。
```java
animator.addListener(new Animator.AnimatorListener() {
    @Override
    public void onAnimationStart(Animator animation) {}
    @Override
    public void onAnimationEnd(Animator animation) {}
    // 其他回调...
});
```

#### 五、实战场景与面试题
​1.**实现一个无限循环的弹性动画**​：
```java
ValueAnimator animator = ValueAnimator.ofFloat(0f, 1f);
animator.setRepeatCount(ValueAnimator.INFINITE);
animator.setInterpolator(new OvershootInterpolator());
animator.addUpdateListener(animation -> {
    float value = (float) animation.getAnimatedValue();
    view.setTranslationY(value * 100);
});
animator.start();
```
2. ​**如何让动画在界面不可见时暂停**​：
    - 监听 `ViewTreeObserver.OnGlobalLayoutListener`，根据 View 可见性控制动画。
3. ​**解释属性动画的 `setDuration()` 和 `setStartDelay()`：
    - `setDuration()`：动画总时长。
    - `setStartDelay()`：动画启动前的延迟时间。

#### 总结

Android 动画考点围绕 ​**分类、实现、原理、优化**​ 展开，需掌握：
1. 补间动画与属性动画的核心区别。
2. 插值器、估值器的自定义实现。
3. 属性动画的内存泄漏规避。
4. 高级动画库（Lottie、矢量动画）的应用。
5. 源码层面的状态机与观察者模式设计。