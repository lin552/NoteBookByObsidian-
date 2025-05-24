---
创建时间: 2025-05-21 18:45:31
作者: wangxiaoming
tags:
  - 设计模式
  - 中介者模式
  - 行为型
---
- **核心思想**​：通过中介对象解耦多个对象间的直接通信。
- ​**场景**​：GUI 组件交互（如按钮、输入框）、微服务 API 网关。

```java
interface Mediator {
    void notify(String event);
}

class ChatRoom implements Mediator {
    private List<User> users = new ArrayList<>();
    public void addUser(User u) { users.add(u); }
    @Override
    public void notify(String event) {
        for (User u : users) u.receive(event);
    }
}
```
