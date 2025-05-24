---
创建时间: 2025-04-20 11:36:22
作者: wangxiaoming
tags:
  - Java
  - 反射
---
#### 一、什么是Java反射
Java 反射（Reflection）是一种在 ​**运行时**​ 动态获取类的完整信息（如类名、属性、方法、构造器等）并操作类的能力。它允许程序像“照镜子”一样检视自身结构，突破编译时静态绑定的限制，实现动态创建对象、调用方法、修改属性等操作
#### 二、核心原理
- ​**Class 对象**​：每个类被 `JVM` 加载时，会生成唯一的 `Class` 对象，存储类的元数据（如字段、方法、构造器等）
- ​**反射 API**​：通过 `java.lang.reflect` 包中的 `Field`（字段）、`Method`（方法）、`Constructor`（构造器）等类，操作这些元数据
- ​**动态性**​：反射绕过了编译时的类型检查，直接与 `JVM` 交互，实现运行时动态行为
#### 三、基础使用
##### 1）获取Class对象
```java
//方式1 类名.class
Class<?> clazz = String.class;
//方式2 对象.getClass()
String s = "hello";
Class<?> clazz = s.getClass();
//方式3 Class.forName()
Class<?> clazz = Class.forName("java.lang.String");
```
##### 2）操作字段和方法
```java
//获取字段
Field field = clazz.getDeclaredField("name");
field.setAccessible(true); //访问私有字段
field.set(obj,"新值");  //修改字段值

//调用方法
Method method = clazz.getMethod("methodName",String.class);
method.invoke(obj,"参数");
```
##### 3）创建对象
```java
//通过构造器
Constructor<?> constructor = clazz.getConstructor(String.class);
Object obj = constructor.newInstance("参数");

//无参构造（需类有无参构造）
Object obj = clazz.newInstance(); //已过时，推荐 getDeclaredConstructor().newInstance()
```
#### 四、高级使用
- ​**动态代理**​：通过 `Proxy` 和 `InvocationHandler` 实现 `AOP`，如日志、事务管理
- ​**注解处理**​：运行时解析注解（如 Spring 的 `@Autowired`）
- ​**泛型擦除**​：通过 `Type` 接口获取泛型实际类型（如 `List<String>` 中的 `String`）
- ​**修改 final 字段**​：通过反射强行修改 `final` 修饰的变量（需谨慎）
- ​**动态加载类**​：结合类加载器，实现插件化架构
#### 五、注意事项
1. ​**性能问题**​：反射操作比直接调用慢约 2-10 倍，频繁调用需优化
2. ​**破坏封装性**​：可访问私有成员，可能导致安全漏洞（如修改敏感数据）
3. ​**异常处理**​：需处理 `ClassNotFoundException`、`NoSuchMethodException` 等异常
4. ​**兼容性**​：反射代码依赖类的内部结构，类结构变动易导致错误
#### 六、性能优化
- ​**缓存反射对象**​：将 `Method`、`Field` 等存入缓存，避免重复获取
- ​**使用 `MethodHandle`**​：Java 7+ 提供更高效的反射调用方式
- ​**避免频繁操作**​：如循环中避免重复调用反射 API
- ​**关闭安全检查**​：`setAccessible(true)` 跳过访问检查（但需权衡安全性）
#### 七、面试考察点
1. ​**反射原理**​：`Class` 对象的作用、反射与动态代理的关系
2. ​**获取 Class 对象的方式**​：三种方法及区别
3. ​**访问私有方法**​：`setAccessible(true)` 的作用
4. ​**反射与封装的关系**​：为何反射可能破坏封装？如何避免？
5. ​**性能优化**​：如何提升反射效率？
#### 八、适用场景
- **框架开发**​：如 Spring 的依赖注入、MyBatis 的 ORM 映射
- ​**动态代理**​：实现 AOP 切面编程
- ​**插件系统**​：运行时加载外部模块（如 IDE 插件）
- ​**测试工具**​：调用私有方法进行单元测试
- ​**配置文件驱动**​：根据配置动态创建对象（如数据库驱动加载）
