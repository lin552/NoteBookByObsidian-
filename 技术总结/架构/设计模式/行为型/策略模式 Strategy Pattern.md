---
创建时间: 2025-05-21 17:18:56
作者: wangxiaoming
tags:
  - 设计模式
  - 策略模式
---
**特点**​：定义算法族，使其可以互相替换。

```java
interface SortStrategy {
    void sort(int[] array);
}

class QuickSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        // 实现快速排序
    }
}

class MergeSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        // 实现归并排序
    }
}

class Sorter {
    private SortStrategy strategy;
    
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void executeSort(int[] array) {
        strategy.sort(array);
    }
}
```
