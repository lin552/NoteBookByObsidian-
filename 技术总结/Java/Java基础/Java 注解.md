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
#### 七、实用注解
##### 1）BeanValidation(JSR 380/Jakarta Bean Validation)
1️⃣ 基本验证注解
```java
import javax.validation.constraints.*;

public class User {
    @NotNull
    private String name;
    
    @Min(18)
    @Max(100)
    private int age;
    
    @Size(min = 6, max = 20)
    private String password;
    
    @Email
    private String email;
    
    @Pattern(regexp = "^[0-9]{11}$")
    private String phone;
    
    @Positive
    private BigDecimal salary;
    
    @Past
    private LocalDate birthDate;
    
    @Future
    private LocalDateTime appointmentTime;
}
```

2️⃣ 方法参数验证
```java
import javax.validation.Valid;
import javax.validation.constraints.*;

public class UserService {
    
    public void createUser(
        @NotBlank(message = "用户名不能为空") String username,
        @Email(message = "邮箱格式不正确") String email,
        @Min(value = 18, message = "年龄必须大于18岁") int age,
        @Valid @NotNull(message = "地址信息必填") Address address
    ) {
        // 方法逻辑
    }
}
```
##### 2) Google Guava的 @Nullable/@Nonnull
```java
import com.google.common.annotations.Nullable;
import org.jetbrains.annotations.NotNull;

public class Example {
    public void processString(@NotNull String input) {
        // 保证input不为null
    }
    
    public String findName(@Nullable String id) {
        // 可能为null的返回值
        if (id == null) {
            return null;
        }
        return database.get(id);
    }
}
```
##### 3)Apache Commons Lang 的 @NonNull
```java
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.Validate;

public class ValidatorExample {
    
    public void validateInput(@org.apache.commons.lang3.builder.ToStringStyle.NonNull String param) {
        Validate.notNull(param, "参数不能为null");
        Validate.notBlank(param, "参数不能为空白");
        Validate.isTrue(param.length() <= 100, "参数长度不能超过100");
    }
}
```
##### 4)Spring Framework的验证注解
```java
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

@RestController
@Validated  // 启用方法级验证
public class UserController {
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable @Min(1) Long id) {
        return userService.findById(id);
    }
    
    @PostMapping("/users")
    public ResponseEntity<?> createUser(@RequestBody @Valid UserDto userDto) {
        // @Valid 触发嵌套对象验证
        return ResponseEntity.ok(userService.create(userDto));
    }
    
    @GetMapping("/search")
    public List<User> searchUsers(
        @RequestParam @Size(min = 1, max = 50) String keyword,
        @RequestParam(defaultValue = "0") @Min(0) int page,
        @RequestParam(defaultValue = "10") @Min(1) @Max(100) int size
    ) {
        return userService.search(keyword, page, size);
    }
}
```
##### 5)自定义类型限制注解
###### 枚举指限制
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumValidator.class)
public @interface ValidEnum {
    Class<? extends Enum<?>> enumClass();
    String message() default "必须是有效的枚举值";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 使用示例
public class Order {
    @ValidEnum(enumClass = OrderStatus.class, message = "订单状态无效")
    private String status;
}
```
###### 数值范围限制
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = RangeValidator.class)
public @interface ValidRange {
    double min() default Double.MIN_VALUE;
    double max() default Double.MAX_VALUE;
    String message() default "数值超出允许范围";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 使用示例
public class Product {
    @ValidRange(min = 0, max = 10000, message = "价格必须在0-10000之间")
    private BigDecimal price;
}
```
##### 6)Lombok的类型安全构建
```java
import lombok.Builder;
import lombok.Value;
import lombok.extern.jackson.Jacksonized;

@Value
@Builder
@Jacksonized
public class ApiRequest {
    @NonNull String userId;           // 自动非空检查
    @NonNull String action;
    @NonNull Map<String, Object> data;
    
    // 编译时会生成包含所有@NonNull检查的构建逻辑
}
```