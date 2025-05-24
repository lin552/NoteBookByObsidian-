---
创建时间: 2025-05-21 17:54:43
作者: wangxiaoming
tags:
  - 设计模式
  - 迭代器模式
  - 行为型
---
- **核心思想**​：统一遍历不同集合对象的方式。
- ​**场景**​：自定义数据结构（如树、图）的遍历。

```java
/**  
 * 迭代器模式 Iterator  
 * * 核心思想：统一遍历不同集合对象的方式。  
 * 场景：自定义数据结构（如树、图）的遍历。  
 *   
* @param <T>  
 */  
public class ListIterator<T> implements Iterator<T> {  
  
    private List<T> list;  
    private int index = 0;  
  
    @Override  
    public boolean hasNext() {  
        return index < list.size();  
    }  
  
    @Override  
    public T next() {  
        return list.get(index++);  
    }  
}
```
