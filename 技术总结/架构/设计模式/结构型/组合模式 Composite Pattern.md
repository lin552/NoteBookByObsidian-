---
创建时间: 2025-05-21 17:46:19
作者: wangxiaoming
tags:
  - 设计模式
  - 组合模式
  - 结构性
---
- ​**核心思想**​：将对象组织成树形结构，统一处理单个对象和组合对象。
- ​**场景**​：文件系统、GUI 组件（如按钮、面板嵌套）。

```java
interface Component {
    void operation();
}

class Leaf implements Component {
    @Override
    public void operation() { /* 叶子节点操作 */ }
}

class Composite implements Component {
    private List<Component> children = new ArrayList<>();
    public void add(Component c) { children.add(c); }
    @Override
    public void operation() {
        for (Component c : children) c.operation();
    }
}
```
