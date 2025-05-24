---
创建时间: 2025-05-21 17:20:14
作者: wangxiaoming
tags:
  - 设计模式
  - 代理模式
---
**特点**​：为其他对象提供代理以控制访问。  
​**示例**​：虚拟代理延迟加载大资源。

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() { 
        System.out.println("Loading " + filename); 
    }
    
    @Override
    public void display() { System.out.println("Displaying " + filename); }
}

class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;
    
    public ProxyImage(String filename) { this.filename = filename; }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}
```
