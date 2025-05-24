---
创建时间: 2025-04-20 11:49:53
作者: wangxiaoming
tags:
  - Java
  - 注解
---
#### 一、什么是注解？
Java 注解（Annotation）是 `JDK5` 引入的元数据机制，用于为代码（类、方法、字段等）添加额外信息。这些信息不直接影响程序逻辑，但可以被编译器、工具或运行时环境读取，实现编译检查、代码生成、框架配置等功能
。例如，`@Override` 标识方法重写父类方法，`@Autowired` 实现依赖注入。
#### 二、核心原理
- ​**本质**​：注解是继承 `java.lang.annotation.Annotation` 的特殊接口，通过 `@interface` 定义
- ​**元注解控制行为**​：
    - `@Retention`：生命周期（SOURCE/CLASS/RUNTIME）
    - `@Target`：作用范围（类、方法、字段等）
    - `@Documented`：生成到 `Javadoc`
    - `@Inherited`：子类继承父类注解
- ​**处理机制**​：
    - ​**编译时处理**​：通过注解处理器（APT）生成代码（如 `Lombok`）
    - ​**运行时处理**​：反射 API 读取注解信息（如 Spring 依赖注入）
#### 三、基础使用
##### 1）内置注解
- `@Override`：校验方法重写
- `@Deprecated`：标记过时元素
- `@SuppressWarnings`：抑制编译器警告
##### 2）自定义注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnnotation {
   String value() default "";
   int priority() default 0;
}
```
##### 3）反射读取
```java
Method method = obj.getClass().getMethod("test");
if(method.isAnnotationPresent(MyAnnotation.class)){
  MyAnnotation anno = method.getAnnotation(MyAnnotation.class);
  System.out.println(anno.value());
}
```
#### 四、高级用法
- ​**动态代理与 `AOP`**​：通过 `@Aspect` 定义切面，结合反射实现日志、事务管理
- ​**注解处理器**​：编译时生成代码（如 `Lombok` 的 `@Data`）
- ​**框架整合**​：
    - `Spring` 的 `@Component`、`@Autowired`
    - `JUnit` 的 `@Test`、`@BeforeEach`
#### 五、注意事项
1. ​**性能开销**​：频繁反射读取 RUNTIME 注解可能降低性能，建议缓存结果
2. ​**破坏封装性**​：通过反射可访问私有字段，需谨慎使用 `setAccessible(true)`
3. ​**兼容性风险**​：注解依赖类结构，修改注解属性可能导致依赖代码出错
4. ​**生命周期匹配**​：确保注解的保留策略（如 RUNTIME）与使用场景一致
#### 六、优化策略
- ​**缓存反射对象**​：将 `Method`、`Field` 缓存以减少重复解析
- ​**编译时处理优先**​：使用 APT 生成代码替代运行时反射（如 `MapStruct`）
- ​**合理设计注解属性**​：避免复杂数据结构，优先使用基本类型和字符串

#### 七、面试考察点
1. ​**原理**​：
    - 元注解的作用及区别（如 `@Retention` vs `@Target`）
    - 注解处理流程（APT 和反射的区别）
2. ​**应用**​：
    - 如何通过反射读取注解？写出代码示例
    - 自定义注解的步骤及注意事项
3. ​**场景**​：
    - Spring 如何利用注解实现依赖注入？
    - `@Override` 在编译时如何校验？
#### 八、适用场景
1. ​**框架配置**​：Spring 的 `@Component`、`@RequestMapping` 简化 XML 配置
2. ​**单元测试**​：`JUnit` 的 `@Test` 标记测试方法
3. ​**代码生成**​：`Lombok` 的 `@Getter` 自动生成方法
4. ​**`AOP` 编程**​：结合 `@Aspect` 实现日志、事务管理
5. ​**数据校验**​：Hibernate 的 `@NotNull` 校验字段合法性
6. ​**文档生成**​：Swagger 的 `@ApiModel` 生成 API 文档
