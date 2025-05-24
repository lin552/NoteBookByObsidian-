---
创建时间: 2025-05-21 17:17:54
作者: wangxiaoming
tags:
  - 设计模式
  - 观察者模式
---
**特点**​：定义对象间的一对多依赖，当一个对象状态变化时通知所有依赖者。

```java
interface Observer {
    void update(String message);
}

class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void notifyObservers(String message) {
        for (Observer o : observers) {
            o.update(message);
        }
    }
}

class User implements Observer {
    private String name;
    
    public User(String name) { this.name = name; }
    
    @Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
```
