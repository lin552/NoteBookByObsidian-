---
创建时间: 2025-05-21 17:35:51
作者: wangxiaoming
tags:
  - 设计模式
  - 外观模式
  - 结构性
---
- ​**核心思想**​：为复杂子系统提供统一的简化接口。
- ​**场景**​：分层架构中封装底层复杂性（如 `SDK` 初始化）。

```java
class SubsystemA { public void operationA() { /* ... */ } }
class SubsystemB { public void operationB() { /* ... */ } }

class Facade {
    private SubsystemA a = new SubsystemA();
    private SubsystemB b = new SubsystemB();
    public void simplifiedOperation() {
        a.operationA();
        b.operationB();
    }
}
```
