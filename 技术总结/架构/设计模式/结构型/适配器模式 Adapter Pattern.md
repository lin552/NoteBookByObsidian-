---
创建时间: 2025-05-21 17:15:22
作者: wangxiaoming
tags:
  - 设计模式
  - 适配器模式
---
**特点**​：将一个类的接口转换为客户端期望的另一个接口。

```java
interface Printer {
    void print(String text);
}

class LegacyPrinter {
    void showText(String text) { System.out.println(text); }
}

class PrinterAdapter implements Printer {
    private LegacyPrinter legacyPrinter;
    
    public PrinterAdapter(LegacyPrinter printer) {
        this.legacyPrinter = printer;
    }
    
    @Override
    public void print(String text) {
        legacyPrinter.showText(text);
    }
}
```
