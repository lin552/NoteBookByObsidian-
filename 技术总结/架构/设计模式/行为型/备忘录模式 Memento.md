---
创建时间: 2025-05-21 18:49:29
作者: wangxiaoming
tags:
  - 设计模式
  - 备忘录模式
  - 行为型
---

- **核心思想**​：保存对象状态以便后续恢复。
- ​**场景**​：撤销/重做操作（如编辑器、游戏存档）

```java
class Memento {
    private String state;
    public Memento(String state) { this.state = state; }
    public String getState() { return state; }
}

class Originator {
    private String state;
    public Memento saveState() { return new Memento(state); }
    public void restoreState(Memento m) { state = m.getState(); }
}
```