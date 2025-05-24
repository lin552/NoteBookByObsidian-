---
创建时间: 2025-04-24 11:11:35
作者: wangxiaoming
tags:
  - ConstraintLayout
---

#### 场景一
三个`TextView` 左边宽度固定（左对齐） 中间内容扩展宽度 右边宽度固定（右对齐）
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <!-- 左 TextView（固定宽度） -->
    <TextView
        android:id="@+id/leftTextView"
        android:layout_width="100dp"  <!-- 固定宽度 -->
        android:layout_height="wrap_content"
        android:text="左固定"
        app:layout_constraintHorizontal_chainStyle="spread_inside"  <!-- 链样式 -->
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>

    <!-- 中 TextView（内容自适应，最大宽度限制） -->
    <TextView
        android:id="@+id/middleTextView"
        android:layout_width="0dp"  <!-- 0dp 表示 MATCH_CONSTRAINT -->
        android:layout_height="wrap_content"
        android:text="中间内容自适应，可能很长很长..."
        app:layout_constraintLeft_toRightOf="@id/leftTextView"  <!-- 左贴左 TextView -->
        app:layout_constraintRight_toLeftOf="@id/rightTextView"  <!-- 右贴右 TextView -->
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintWidth_default="wrap"  <!-- 宽度按内容扩展 -->
        app:layout_constraintWidth_max="0dp"  <!-- 最大宽度为剩余空间（需配合 0dp）[7](@ref)"/>) 

    <!-- 右 TextView（固定宽度） -->
    <TextView
        android:id="@+id/rightTextView"
        android:layout_width="100dp"  <!-- 固定宽度 -->
        android:layout_height="wrap_content"
        android:text="右固定"
        app:layout_constraintRight_toRightOf="parent"  <!-- 右对齐 -->
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```
#### 场景二
三个 `TextView` 左边宽度固定 中间内容扩展宽度 右边内容扩展宽度 （所有左对齐）
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <!-- 左 TextView（固定宽度） -->
    <TextView
        android:id="@+id/leftTextView"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="左固定"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>

    <!-- 中间 TextView（内容自适应，左对齐） -->
    <TextView
        android:id="@+id/middleTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="中间内容自适应"
        app:layout_constraintLeft_toRightOf="@id/leftTextView"
        app:layout_constraintRight_toLeftOf="@id/rightTextView"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintWidth_default="wrap"
        app:layout_constraintHorizontal_bias="0"/>

    <!-- 右 TextView（内容自适应，左对齐） -->
    <TextView
        android:id="@+id/rightTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="右内容自适应"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintLeft_toRightOf="@id/middleTextView"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintWidth_default="wrap"
        app:layout_constraintHorizontal_bias="0"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```