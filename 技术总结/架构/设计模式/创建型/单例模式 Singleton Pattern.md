---
创建时间: 2025-05-21 17:06:08
作者: wangxiaoming
tags:
  - 设计模式
  - 单例模式
  - 创建型
---
#### 一、懒汉式 (Lazy Loading)
**特点**​：延迟实例化，首次调用时创建实例。  
​**问题**​：线程不安全（未同步）。

```java
/**  
 * 懒汉式 单例模式  
 *  
 * 特点：延迟实例化，首次调用时创建实例。  
 * 问题：线程不安全（未同步）。  
 */  
public class LazySingleton {  
    private static LazySingleton instance;  
    
    private LazySingleton() {  
    }  
  
    public static synchronized LazySingleton getInstance() {  
        if (instance == null) {  
            instance = new LazySingleton();  
        }  
        return instance;  
    }  
}
```

#### 二、双重检查锁 (Double-Checked Locking)
**特点**​：线程安全且高效，延迟加载。  
​**关键点**​：`volatile` 防止指令重排序，双重检查减少同步开销。

```java
/**  
 * 双重检查锁 Double-Checked Locking  
 * * 特点：线程安全且高效，延迟加载。  
 * 关键点：volatile 防止指令重排序，双重检查减少同步开销。  
 */  
public class DCLSingleton {  
    private static DCLSingleton instance;  
  
    private DCLSingleton() {  
  
    }  
  
    public static DCLSingleton getInstance() {  
        if (instance == null) {  
            synchronized (DCLSingleton.class) {  
                if (instance == null) {  
                    instance = new DCLSingleton();  
                }  
            }  
        }  
        return instance;  
    }  
}
```

#### 三、静态内部类 (Holder)
​**特点**​：利用类加载机制保证线程安全，延迟加载。  
​**原理**​：内部类在第一次被引用时加载，由 `JVM` 保证线程安全。

```java
/**  
 * 静态内部类 Holder  
 * * 特点：利用类加载机制保证线程安全，延迟加载。  
 * 原理：内部类在第一次被引用时加载，由 JVM 保证线程安全。  
 */  
public class HolderSingleton {  
    private HolderSingleton() {}  
  
    private static class Holder {  
        static final HolderSingleton INSTANCE = new HolderSingleton();  
    }  
  
    public static HolderSingleton getInstance() {  
        return Holder.INSTANCE;  
    }  
}
```

#### 四、枚举单例 （`Enum Singleton`)
**特点**​：绝对线程安全，防止反射攻击，推荐方式。  
​**原理**​：枚举实例在 `JVM` 中唯一且不可变。

```java
/**  
 * 枚举单例 Enum Singleton  
 * * 特点：绝对线程安全，防止反射攻击，推荐方式。  
 * 原理：枚举实例在 JVM 中唯一且不可变。  
 */  
public enum EnumSingleton {  
    INSTANCE;  
  
    public void doSomething() {  
        System.out.println("EnumSingleton working");  
    }  
}
```