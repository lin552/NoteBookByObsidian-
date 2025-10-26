---
创建时间: 2025-06-06 01:05:00
作者: wangxiaoming
tags:
  - Dagger2
---

`Dagger2` 是 Google 推出的依赖注入框架，以**编译时生成代码**为核心，解决模块间耦合问题，提升代码可维护性和可测试性。其核心考点围绕**依赖注入机制**、**模块化设计**、**作用域管理**及**性能优化**展开，以下是系统梳理：

#### 一、核心组件与工作原理
##### 1. 核心注解​
- ​**`@Inject`**​
    - ​**作用**​：标记需要注入的依赖（字段、构造函数或方法）。
    - ​**示例**​：
        ```java
        public class User {
            @Inject // 构造函数注入
            public User() {}
        }
        ```

- ​**`@Module`**​
    - ​**作用**​：提供依赖实例的类，通过 `@Provides` 注解方法返回依赖。
    - ​**示例**​：
        ```java
        @Module
        public class AppModule {
            @Provides
            public User provideUser() {
                return new User();
            }
        }
        ```

- ​**`@Component`**​
    - ​**作用**​：连接 `@Module` 和 `@Inject` 的桥梁，定义注入方法。
    - ​**示例**​：
        ```java
        @Component(modules = AppModule.class)
        public interface AppComponent {
            void inject(MainActivity activity);
        }
        ```

##### ​2. 依赖注入流程​
1. ​**编译时生成代码**​：APT（注解处理器）根据 `@Inject` 和 `@Provides` 生成工厂类（如 `User_Factory`）和组件类（如 `DaggerAppComponent`）
2. ​**运行时注入**​：通过 `Component.inject()` 方法将依赖注入目标类。

##### ​3. 依赖解析规则​
- ​**优先级**​：`@Module` 提供的依赖优先级高于 `@Inject` 构造函数。
- ​**多模块依赖**​：通过 `@Component(modules = {A.class, B.class})` 引入多个模块，支持模块间依赖（`dependencies` 参数）

#### 二、依赖注入类型
##### 1. 构造函数注入​
- ​**强制依赖**​：通过 `@Inject` 标记构造函数，确保依赖在对象创建时明确。
    ```java
    public class Repository {
        private final ApiService apiService;
        @Inject
        public Repository(ApiService apiService) {
            this.apiService = apiService;
        }
    }
    ```

##### ​2. 字段注入​
- ​**灵活但隐藏依赖**​：直接在字段上标注 `@Inject`，但需注意可读性。
    ```java
    public class Activity {
        @Inject
        Repository repository; // 依赖由 Dagger 自动注入
    }
    ```

##### ​3. 方法注入​
- ​**动态依赖**​：通过 `@Inject` 标记方法参数，适用于运行时动态确定依赖。
    ```java
    public class Presenter {
        @Inject
        void init(ApiService apiService) { // 依赖通过参数注入
            // ...
        }
    }
    ```
#### 三、作用域管理
##### 1. 单例（`@Singleton`）​​
- ​**全局单例**​：在 `@Provides` 方法和 `@Component` 上标注 `@Singleton`，确保同一组件内单例。
    ```java
    @Module
    public class AppModule {
        @Singleton
        @Provides
        public Database provideDatabase() {
            return new Database();
        }
    }
    ```

##### ​2. 自定义作用域​
- ​**限定生命周期**​：通过自定义注解（如 `@ActivityScope`）控制依赖作用域。
    ```java
    @Scope
    @Retention(RetentionPolicy.RUNTIME)
    public @interface ActivityScope {}
    
    @Component(modules = ActivityModule.class)
    @ActivityScope
    public interface ActivityComponent {
        void inject(Activity activity);
    }
    ```
#### 四、模块化与多组件设计
##### 1. 模块依赖​
- ​**`@Binds` 简化接口绑定**​：将接口与实现类绑定，减少冗余代码。
    ```java
    @Module
    public abstract class RepositoryModule {
        @Binds
        abstract DataSource bindDataSource(DataSourceImpl impl);
    }
    ```

##### ​2. 子组件（`@Subcomponent`）​​
- ​**层次化管理**​：子组件继承父组件的依赖，实现模块化分层。
    ```java
    @Subcomponent(modules = FeatureModule.class)
    public interface FeatureComponent {
        void inject(FeatureActivity activity);
    }
    ```

#### 五、高频面试题
1. ​`Dagger2` 如何解决依赖注入？​**​
    - 通过 `@Inject` 标记依赖，`@Module` 提供实例，`@Component` 连接两者，编译时生成代码完成注入
2. ​`Dagger2` 与 `Dagger1` 的区别？​**​
    - `Dagger2` 使用编译时生成代码，避免反射，性能更高；`Dagger1` 依赖运行时反射，灵活性差
3. ​**如何实现单例？​**​
    - 在 `@Provides` 方法和 `@Component` 上标注 `@Singleton`，确保同一组件内单例
4. ​**模块间依赖如何处理？​**​
    - 通过 `@Component(dependencies = ParentComponent.class)` 引用父组件，或使用 `includes` 引入其他模块
5. ​`Dagger2 与 Hilt 的关系？​**​
    - Hilt 是 `Dagger2` 的 Android 优化版，简化配置（如自动绑定 `Application` 上下文），更适合 Android 开发
#### 六、源码解析
##### 1. 生成的代码结构​
- ​**`DaggerComponent`**​：自动生成的组件类，管理依赖实例。
- ​**`Factory` 类**​：如 `User_Factory`，负责创建依赖实例。
- ​**`MembersInjector` 类**​：如 `MainActivity_MembersInjector`，注入字段或方法
##### ​2. 依赖解析流程​
1. ​**构建依赖图**​：APT 解析 `@Inject` 和 `@Provides`，生成依赖关系图。
2. ​**生成代码**​：根据依赖图生成工厂和组件类。
3. ​**运行时注入**​：通过 `Component.inject()` 调用生成的代码完成注入。

#### 七、最佳实践
1. ​**避免循环依赖**​：通过接口或 `Lazy<T>` 解耦循环依赖。
2. ​**使用 `@Named` 区分同名依赖**​：
    ```java
    @Provides
    @Named("dev")
    public Api provideDevApi() { ... }
    
    @Provides
    @Named("prod")
    public Api provideProdApi() { ... }
    ```
3. ​**单元测试友好**​：通过替换 `@Module` 实现 Mock 依赖。