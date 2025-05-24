---
创建时间: 2025-04-12 17:34:32
作者: wangxiaoming
tags:
  - Java
  - 多线程
  - 原子类
  - CAS
---
**实现机制**：
所有原子类均通过`Unsafe`类的`CAS`操作实现原子性，如`AtomicInteger`的`compareAndSet()`方法调用`Unsafe.compareAndSwapInt()`

#### 一、基础原子类
##### 1)`AtomicInteger/AtomicLong`
针对整型和长整型的原子操作，支持自增、比较交换等方法
```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // 原子自增
```
##### 2)`AtomicBoolean`
布尔型原子变量，常用于状态标志位

#### 二、原子引用类

##### 1)`AtomicReference`
对对象引用的原子操作

##### 2)`AtomicStampedReference`
通过版本号解决ABA问题
```java
AtomicStampedReference<Integer> ref = new AtomicStampedReference<>(1, 0);
ref.compareAndSet(1, 2, 0, 1);  // 检查值及版本号
```

#### 三、高并发优化类

##### 1)`LongAdder`
分段累加，适用于高竞争场景（如计数器），性能优于`AtomicLong`
##### 2)`DoubleAdder`
针对双精度浮点型的优化类

#### 四、数组与字段原子类

##### 1）`AtomicIntegerArray`
原子操作整型数组元素。
#### 2)`AtomicReferenceFieldUpdater`
原子更新对象中的字段

