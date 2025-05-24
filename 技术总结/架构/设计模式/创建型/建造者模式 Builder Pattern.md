---
创建时间: 2025-05-21 17:13:44
作者: wangxiaoming
tags:
  - 设计模式
  - 建造者模式
  - 创建型
---
**特点**​：分步构造复杂对象，分离对象的构造与表示。  
​**适用场景**​：对象参数多且可选（如 SQL 查询、HTTP 请求）。

```java
/**  
 * 建造者模式 builder Pattern  
 * * 特点：分步构造复杂对象，分离对象的构造与表示。  
 * 适用场景：对象参数多且可选（如 SQL 查询、HTTP 请求）。  
 */  
public class Pizza {  
    private String dough;  
    private String sauce;  
    private String topping;  
  
    public Pizza(Builder builder) {  
        this.dough = builder.dough;  
        this.sauce = builder.sauce;  
        this.topping = builder.topping;  
    }  
  
    public static class Builder {  
        private String dough;  
        private String sauce;  
        private String topping;  
  
        public Builder setDough(String dough) {  
            this.dough = dough;  
            return this;  
        }  
  
        public Builder setSauce(String sauce) {  
            this.sauce = sauce;  
            return this;  
        }  
  
        public Builder setTopping(String topping) {  
            this.topping = topping;  
            return this;  
        }  
  
        public Pizza build() {  
            return new Pizza(this);  
        }  
    }  
  
    /**  
     * 使用  
     * @param args  
     */  
    public static void main(String[] args) {  
        Pizza pizza = new Pizza.Builder()  
                .setDough("thin")  
                .setSauce("tomato")  
                .setTopping("cheese")  
                .build();  
    }  
}
```
