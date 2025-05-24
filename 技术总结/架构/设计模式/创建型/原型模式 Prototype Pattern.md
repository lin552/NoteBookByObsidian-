---
创建时间: 2025-05-21 17:33:51
作者: wangxiaoming
tags:
  - 设计模式
  - 创建型
  - 原型模式
---
- **核心思想**​：通过克隆现有对象创建新对象，避免重复初始化。
- ​**场景**​：对象创建成本高（如数据库查询）、需要动态生成对象。

```java
interface Prototype extends Cloneable {
    Prototype clone();
}

class Sheep implements Prototype {
    private String name;
    @Override
    public Sheep clone() {
        try {
            return (Sheep) super.clone();
        } catch (CloneNotSupportedException e) {
            return new Sheep(name); // 简单实现
        }
    }
}
```
