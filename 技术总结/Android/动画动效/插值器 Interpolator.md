---
创建时间: 2026-02-03 10:18:28
作者: wangxiaoming
tags:
---
#### 一、概念和作用

插值器(Interpolator)定义了动画的变化率,控制动画在时间轴上的速度曲线,使动画更加自然流畅。
#### 二、常用插值器
##### 1）`LinearInterpolator` - 线性匀速
- 线性匀速运动
- 动画从头到尾速度不变

```java
// 匀速移动动画  
ValueAnimator animator = ValueAnimator.ofFloat(0f, 100f);  
animator.setDuration(1000);  
animator.setInterpolator(new LinearInterpolator());  
animator.addUpdateListener(animation -> {  
    float value = (float) animation.getAnimatedValue();  
    view.setTranslationX(value);  
});  
animator.start();
```
##### 2) `AccelerateInterpolator` - 加速
- 加速运动
- 开始慢,逐渐加快

```java
// 视图从左侧加速滑入  
ObjectAnimator animator = ObjectAnimator.ofFloat(view, "translationX", -500f, 0f);  
animator.setDuration(500);  
animator.setInterpolator(new AccelerateInterpolator(1.5f)); // 参数控制加速度  
animator.start();
```
##### 3)`DecelerateInterpolator` - 减速
- 减速运动
- 开始快,逐渐减慢

```java
// 小播放器滑动停靠,越靠近目标越慢  
ValueAnimator animator = ValueAnimator.ofInt(currentBottomMargin, targetBottomMargin);  
animator.setDuration(300);  
animator.setInterpolator(new DecelerateInterpolator(2.0f)); // 参数越大减速越明显  
animator.addUpdateListener(animation -> {  
    int value = (int) animation.getAnimatedValue();  
    LayoutParams params = (LayoutParams) smallPlayerView.getLayoutParams();  
    params.bottomMargin = value;  
    smallPlayerView.setLayoutParams(params);  
});  
animator.start();
```
##### 4）`AccelerateDecelerateInterpolator` - 先加速后减速
- 先加速后减速
- Android 默认插值器
- 中间快,两头慢
```java
// 视图平滑切换位置(最常用)  
ObjectAnimator animator = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f);  
animator.setDuration(400);  
animator.setInterpolator(new AccelerateDecelerateInterpolator());  
animator.start();
```
##### 5）`AnticipateInterpolator` - 先后退再前进
- 先后退再前进
- 适合弹性效果
```java
// 按钮点击先缩小再放大  
ObjectAnimator animator = ObjectAnimator.ofFloat(button, "scaleX", 1f, 1.2f);  
animator.setDuration(300);  
animator.setInterpolator(new AnticipateInterpolator(2.0f)); // 参数控制后退幅度  
animator.start();
```
##### 6) `OvershootInterpolator` -  超出后回弹
- 超出目标值后回弹
- 结束时会略微超出
```java
// 列表项展开动画,略微超出再回弹  
ValueAnimator animator = ValueAnimator.ofInt(0, targetHeight);  
animator.setDuration(500);  
animator.setInterpolator(new OvershootInterpolator(1.5f)); // 参数控制超出幅度  
animator.addUpdateListener(animation -> {  
    int height = (int) animation.getAnimatedValue();  
    view.getLayoutParams().height = height;  
    view.requestLayout();  
});  
animator.start();
```
##### 7) `BounceInterpolator` - 弹跳
- 弹跳效果
- 结束时产生反弹
```java
// 下拉刷新回弹效果  
ObjectAnimator animator = ObjectAnimator.ofFloat(refreshView, "translationY", -200f, 0f);  
animator.setDuration(600);  
animator.setInterpolator(new BounceInterpolator());  
animator.start();
```
##### 8) `CycleInterpolator` - 循环摆动
- 循环运动
- 按正弦曲线循环
```java
// 密码错误时抖动提示  
ObjectAnimator animator = ObjectAnimator.ofFloat(passwordField, "translationX", 0f, 25f);  
animator.setDuration(500);  
animator.setInterpolator(new CycleInterpolator(3)); // 参数为循环次数  
animator.start();
```
#### 三、自定义插值器
```java
public class EaseCubicInterpolator implements TimeInterpolator {  
    @Override  
    public float getInterpolation(float input) {  
        // 三次贝塞尔曲线  
        if (input < 0.5f) {  
            return 4 * input * input * input;  
        } else {  
            float f = (2 * input) - 2;  
            return 0.5f * f * f * f + 1;  
        }  
    }  
}
```

