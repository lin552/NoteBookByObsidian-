---
创建时间: 2025-05-21 17:58:06
作者: wangxiaoming
tags:
  - 设计模式
  - 责任链模式
  - 行为型
---
- **核心思想**​：将请求沿处理链传递，直到某个对象处理它。
- ​**场景**​：日志处理（多级日志级别）、中间件管道（如 `Express.js`）。

```java
abstract class Handler {
    private Handler next;
    public void setNext(Handler next) { this.next = next; }
    public void handleRequest(Request request) {
        if (next != null) next.handleRequest(request);
    }
}

class AuthHandler extends Handler {
    @Override
    public void handleRequest(Request request) {
        if (request.hasValidToken()) super.handleRequest(request);
        else System.out.println("Unauthorized");
    }
}
```