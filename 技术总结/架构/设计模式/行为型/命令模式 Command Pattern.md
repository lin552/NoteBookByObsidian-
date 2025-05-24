---
创建时间: 2025-05-21 17:50:34
作者: wangxiaoming
tags:
  - 设计模式
  - 命令模式
  - 行为型
---
- **核心思想**​：将请求封装为对象，支持撤销、重做、队列化。
- ​**场景**​：菜单系统、事务操作、宏命令。

```java
interface Command {
    void execute();
    void undo();
}

class LightOnCommand implements Command {
    private Light light;
    @Override
    public void execute() { light.turnOn(); }
    @Override
    public void undo() { light.turnOff(); }
}
```
