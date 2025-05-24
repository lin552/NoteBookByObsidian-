---
创建时间: 2025-05-21 18:36:18
作者: wangxiaoming
tags:
  - 设计模式
  - 模板方法模式
  - 行为型
---

- **核心思想**​：定义算法骨架，允许子类重写特定步骤。
- ​**场景**​：框架中固定流程的扩展（如 Spring 的 `JdbcTemplate`）

```java
abstract class Game {
    abstract void initialize();
    abstract void startPlay();
    // 模板方法
    public final void play() {
        initialize();
        startPlay();
        endPlay();
    }
    void endPlay() { System.out.println("Game Over"); }
}
```