---
创建时间: 2025-05-21 18:37:05
作者: wangxiaoming
tags:
  - 设计模式
  - 状态模式
  - 行为型
---

- **核心思想**​：对象行为随状态改变而改变。
- ​**场景**​：状态机（如订单状态流转、`TCP` 连接状态）

```java
interface State {
    void handle();
}

class Order {
    private State state;
    public void setState(State state) { this.state = state; }
}

class PaidState implements State {
    @Override
    public void handle() { /* 处理已支付逻辑 */ }
}
```