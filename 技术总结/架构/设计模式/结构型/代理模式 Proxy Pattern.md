---
创建时间: 2025-05-21 17:20:14
作者: wangxiaoming
tags:
  - 设计模式
  - 代理模式
---
**特点**​：通过代理对象控制对原对象（真实主题）的访问，从而在不修改原对象的前提下增强功能（如日志记录、权限校验、延迟加载等）。根据代理对象的**生成时机和方式**，代理模式可分为 ​**静态代理**​ 和 ​**动态代理**，两者的核心区别在于代理类的创建方式和灵活性。  
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

#### 二、动态代理和静态代理的区别
|**维度**​|​**静态代理**​|​**动态代理**​|
|---|---|---|
|​**生成时机**​|编译期（固定）|运行期（动态生成）|
|​**实现方式**​|手动编写代理类（强绑定接口）|反射/字节码生成（通用适配）|
|​**灵活性**​|低（仅适用于特定接口）|高（适配多接口，通用逻辑）|
|​**代码冗余**​|高（每个接口需独立代理类）|低（统一处理逻辑）|
|​**典型场景**​|简单功能增强（如单一接口的日志）|AOP、框架扩展（如Spring事务管理）|
|​**性能**​|略高（无反射开销）|略低（反射调用，但现代JVM优化后可忽略）|
##### 详细解析
###### ①静态代理
静态代理的代理类在**编译期**就已经确定，需要开发者手动编写或通过工具生成。代理类与真实主题（被代理对象）实现**相同的接口**，并在调用真实主题方法的前后添加额外逻辑（如日志、权限校验）。

**关键特点：​**​
- ​**手动编写代理类**​：需显式定义代理类，实现与真实主题一致的接口，并在方法中调用真实主题的同名方法，同时添加自定义逻辑。
- ​**强绑定性**​：代理类与接口一一对应，若接口新增方法，代理类需手动修改。
- ​**简单直接**​：逻辑清晰，易于调试，但扩展性和复用性差。
```java
// 接口
interface UserService {
    void addUser(String name);
    void deleteUser(String name);
}

// 真实主题（被代理对象）
class RealUserService implements UserService {
    public void addUser(String name) {
        System.out.println("添加用户：" + name);
    }
    public void deleteUser(String name) {
        System.out.println("删除用户：" + name);
    }
}

// 静态代理类（手动编写）
class StaticProxyUserService implements UserService {
    private final UserService realService;

    public StaticProxyUserService(UserService realService) {
        this.realService = realService;
    }

    // 代理方法：添加日志
    public void addUser(String name) {
        System.out.println("[日志] 开始添加用户");
        realService.addUser(name); // 调用真实主题方法
        System.out.println("[日志] 结束添加用户");
    }

    // 代理方法：添加日志（需重复编写）
    public void deleteUser(String name) {
        System.out.println("[日志] 开始删除用户");
        realService.deleteUser(name); // 调用真实主题方法
        System.out.println("[日志] 结束删除用户");
    }
}

// 使用静态代理
public class StaticProxyDemo {
    public static void main(String[] args) {
        UserService realService = new RealUserService();
        UserService proxy = new StaticProxyUserService(realService);
        proxy.addUser("张三"); // 输出带日志的操作
        proxy.deleteUser("李四");
    }
}
```
**缺点：​**​  
若接口有10个方法，代理类需为每个方法重复编写日志逻辑，代码冗余严重；若接口变更（如新增方法），代理类需手动修改，维护成本高

##### ②动态代理
动态代理的代理类在**运行期**动态生成，无需手动编写。通过反射机制或字节码技术（如`JDK Proxy`、`CGLIB`），在程序运行时为目标对象生成代理对象，代理对象会拦截所有方法调用，并统一处理额外逻辑（如`AOP`中的切面）。

**关键特点：​**​
- ​**动态生成**​：利用Java的`Proxy`类或`CGLIB`库，在运行时根据接口或类动态生成代理类。
- ​**统一处理**​：通过`InvocationHandler`（`JDK`）或`MethodInterceptor`（`CGLIB`）回调接口，集中处理所有方法的调用逻辑，避免重复代码。
- ​**高灵活性**​：可适配多个接口，无需为每个接口编写代理类，适合通用场景（如框架扩展）。
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 通用InvocationHandler（集中处理日志逻辑）
class LogInvocationHandler implements InvocationHandler {
    private final Object realObject; // 真实主题对象

    public LogInvocationHandler(Object realObject) {
        this.realObject = realObject;
    }

    // 拦截所有方法调用
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("[日志] 开始执行方法：" + method.getName());
        Object result = method.invoke(realObject, args); // 调用真实主题方法
        System.out.println("[日志] 结束执行方法：" + method.getName());
        return result;
    }
}

// 使用动态代理
public class DynamicProxyDemo {
    public static void main(String[] args) {
        UserService realService = new RealUserService();
        
        // 动态生成代理对象（需指定类加载器、接口、InvocationHandler）
        UserService proxy = (UserService) Proxy.newProxyInstance(
            UserService.class.getClassLoader(),
            new Class<?>[]{UserService.class},
            new LogInvocationHandler(realService)
        );

        proxy.addUser("张三"); // 输出带日志的操作
        proxy.deleteUser("李四");
    }
}
```
**优点：​**​  
通过统一的`InvocationHandler`处理所有方法调用，避免了静态代理的代码冗余；新增接口方法时，无需修改代理逻辑，只需真实主题实现接口即可自动生效。
