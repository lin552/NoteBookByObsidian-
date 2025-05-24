---
创建时间: 2025-05-21 17:17:10
作者: wangxiaoming
tags:
  - 设计模式
  - 装饰器模式
---
​**特点**​：动态地为对象添加额外职责。

```java
interface Coffee {
    double cost();
    String description();
}

class SimpleCoffee implements Coffee {
    @Override
    public double cost() { return 1; }
    
    @Override
    public String description() { return "Coffee"; }
}

abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
}

class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double cost() { return decoratedCoffee.cost() + 0.5; }
    
    @Override
    public String description() { return decoratedCoffee.description() + ", Milk"; }
}
```
