---
创建时间: 2025-05-21 17:40:25
作者: wangxiaoming
tags:
  - 设计模式
  - 享元模式
  - 结构性
---
- ​**核心思想**​：共享大量细粒度对象，减少内存占用。
- ​**场景**​：高频创建相似对象（如游戏中的粒子系统、文本编辑器字符渲染）。

```java
interface CharacterFlyweight {
    void render(String font);
}

class ConcreteCharacter implements CharacterFlyweight {
    private char c;
    public ConcreteCharacter(char c) { this.c = c; }
    @Override
    public void render(String font) { /* 使用共享的字符数据 */ }
}

class FlyweightFactory {
    private Map<Character, CharacterFlyweight> pool = new HashMap<>();
    public CharacterFlyweight getCharacter(char c) {
        return pool.computeIfAbsent(c, k -> new ConcreteCharacter(k));
    }
}
```
