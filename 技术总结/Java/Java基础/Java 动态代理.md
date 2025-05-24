---
创建时间: 2025-04-20 12:01:43
作者: wangxiaoming
tags:
  - Java
  - 动态代理
---
#### 一、什么是动态代理？
动态代理是Java中一种在**运行时**动态生成代理对象的技术，允许在不修改原始类代码的前提下，对目标对象的方法调用进行拦截和增强。通过代理对象，可以在方法执行前后插入额外逻辑（如日志、事务、权限校验等），实现与业务逻辑的解耦
#### 二、核心原理
- ​**基于反射与接口**​：
    - ​**Proxy类**​：通过`java.lang.reflect.Proxy`动态生成代理类，该类实现目标接口的所有方法
    - ​**`InvocationHandler`接口**​：代理对象的方法调用会被转发到`InvocationHandler.invoke()`方法，开发者在此方法中实现增强逻辑
- ​**代理类的生成**​：
    - `JDK`动态代理通过反射机制生成代理类的字节码，其类名通常为`$Proxy0`、`$Proxy1`等
    - ​**限制**​：只能代理接口，无法直接代理类（需通过`CGLIB`等第三方库）
#### 三、使用方法
步骤示例
##### 1）定义接口与实现类
```java
public interface UserService {  
    void save();  
}

public class UserServiceImpl implements UserService {  
    @Override  
    public void save() {  
  
    }  
}
```
##### 2）实现`InvocationHandler`
```java
public class LogHandler implements InvocationHandler {  
    private Object target; //目标对象  
    public LogHandler(Object target) {  
        this.target = target;  
    }  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
        System.out.println("方法执行前----记录日志");  
        Object result = method.invoke(target, args);//调用目标方法  
        System.out.println("方法执行后----资源释放");  
        return result;  
    }  
}
```
##### 3）生成代理对象
```java
    public static void main(String[] args) {  
        UserService userService = new UserServiceImpl();  
        UserService proxyInstance = (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),      userService.getClass().getInterfaces(), new LogHandler(userService));  
        proxyInstance.save(); //调用代理方法  
    }  
```
#### 四、注意事项
1. ​**性能开销**​：反射调用比直接调用慢，频繁使用需优化
2. ​**接口依赖**​：`JDK`动态代理必须基于接口，若需代理类需用`CGLIB`
3. ​**方法重复调用**​：代理对象调用自身方法可能导致死循环，需通过`method.invoke(target)`避免
4. ​**线程安全**​：`InvocationHandler`需确保线程安全，避免共享状态

#### 五、优化策略
1. ​**缓存代理类**​：避免重复生成代理类字节码，减少反射开销
2. ​**减少反射调用**​：预加载`Method`对象并缓存，避免频繁调用`getMethod()`
3. ​**结合`CGLIB`**​：对无接口的类使用`CGLI`B（基于`ASM`字节码生成）提升性能
4. ​**异步处理**​：耗时操作（如日志写入）异步执行，避免阻塞主线程

#### 六、面试考察点
1. **实现原理**​：反射机制、代理类生成流程
2. ​**与静态代理的区别**​：动态代理无需硬编码代理类，灵活性更高
3. ​**`JDK`与`CGLIB`对比**​：
    - ​`JDK`：基于接口，性能较低
    - ​`CGLIB`：基于继承，支持类代理，性能更高但生成代理类较慢
4. ​**应用场景**​：`AOP`、`RPC`、事务管理等
5. ​**常见陷阱**​：`final`方法无法代理、循环调用问题
#### 七、适用场景
1. ​`AOP`（面向切面编程）​​：如Spring的声明式事务、日志切面
2. ​**远程调用（`RPC`）​**​：隐藏网络通信细节，如`Dubbo`的消费者代理
3. ​**延迟加载**​：`Hibernate`的懒加载机制，访问属性时才触发数据库查询
4. ​**安全控制**​：方法调用前的权限校验
5. ​**日志与监控**​：统计方法执行时间、调用次数

#### 八、何时使用动态代理？
- ​**需无侵入增强功能**​：如为已有系统添加日志，无需修改原代码
- ​**代理多个接口**​：动态代理支持同时代理多个接口，简化代码
- ​**框架扩展**​：如`Spring`的`Bean`动态代理实现依赖注入

