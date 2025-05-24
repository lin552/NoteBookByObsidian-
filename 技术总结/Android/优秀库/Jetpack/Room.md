---
创建时间: 2025-04-21 11:48:55
作者: wangxiaoming
tags:
  - Jetpack
  - Room
---

#### 一、`Room`是什么？
`Room` 是 `Android Jetpack` 中基于 SQLite 的**持久化库**，提供类型安全、编译时校验的数据库抽象层，旨在简化 SQLite 操作并提升开发效率。核心价值包括：
- ​`ORM` 支持​：通过注解将 `Java/Kotlin` 对象映射为数据库表，避免手动处理 SQL 语句。
- ​**编译时验证**​：检查 `SQL` 语法错误，确保查询正确性
- ​**与 `Jetpack` 生态集成**​：无缝结合 `LiveData`、`Flow`、`ViewModel`，支持响应式数据流

#### 二、核心组件与原理
##### 1）三大组件​
- ​**Entity**​：通过 `@Entity` 注解定义表结构，字段对应列，支持主键（`@PrimaryKey`）、索引（`@Index`）和外键（`@ForeignKey`）
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true) val id: Int,
    @ColumnInfo(name = "user_name") val name: String
)
```
- ​`DAO`​：数据访问对象，通过 `@Dao` 接口定义增删改查方法，支持 SQL 查询（`@Query`）及事务（`@Transaction`）
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): Flow<List<User>>
    @Insert
    suspend fun insert(user: User)
}
```
- **Database**​：继承 `RoomDatabase` 的抽象类，通过 `@Database` 注解声明数据库版本及包含的 Entity
##### 2） ​**底层原理**​
- **代码生成**​：编译时生成 `_Impl` 类实现 `DAO` 接口，将注解转换为 SQL 操作
- ​**事务管理**​：通过 `SupportSQLiteDatabase` 封装事务，确保原子性操作
- ​**线程优化**​：默认禁止主线程操作，推荐结合协程或 `RxJava` 实现异步

#### 三、基础与高级用法
##### 1）基础使用步骤
- 添加依赖
```gradle
implementation "androidx.room:room-runtime:2.4.2"
kapt "androidx.room:room-compiler:2.4.2"
```
- ​**定义 `Entity`、`DAO`、`Database`**​（见上文）。
- ​**构建数据库实例**​：
```kotlin
val db = Room.databaseBuilder(context, AppDatabase::class.java, "my-db")
    .fallbackToDestructiveMigration() // 破坏性迁移（仅开发环境）
    .build()
```
##### 2）高级功能
- ​**数据库迁移**​：通过 `Migration` 类处理表结构变更，保留旧数据
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE users ADD COLUMN age INTEGER")
    }
}
```
- ​**响应式查询**​：`DAO` 返回 `Flow` 或 `LiveData`，实现数据变化自动刷新`UI`
- ​**复杂类型处理**​：使用 `@TypeConverter` 转换非基本类型（如 Date、List）

#### 四、注意事项与优化策略
1. ​**常见问题**​
    - ​**主线程阻塞**​：避免在主线程执行耗时操作，使用协程（`suspend`）或异步任务
    - ​**内存泄漏**​：在 Fragment 中通过 `viewLifecycleOwner` 观察 `LiveData`，及时释放资源
    - ​**索引优化**​：为高频查询字段添加索引（`@Index`），提升查询速度
2. ​**性能优化**​
    - ​**分页查询**​：结合 Paging 3 库分批加载数据，避免 `OOM`
    - ​**批量操作**​：使用 `@Insert`、`@Update` 的批量方法减少事务开销
    - ​**缓存策略**​：合理使用 `@Query` 的缓存机制，避免重复查询

#### 五、面试高频考点
1. ​**Room vs SQLite**​
    - ​**开发效率**​：Room 通过注解和编译时检查减少手写 SQL 的错误
    - ​**类型安全**​：`DAO` 方法返回类型明确，避免运行时类型转换异常
2. ​`LiveData/Flow` 集成**​
    - ​**数据观察**​：`DAO` 返回 `LiveData` 或 `Flow`，实现 `UI` 自动更新
    - ​**线程切换**​：Room 自动在后台线程执行查询，主线程更新 `UI`
3. ​**数据库迁移**​
    - ​**版本升级**​：通过 `Migration` 或 `fallbackToDestructiveMigration` 处理结构变更
4. ​**事务与性能**​
    - ​**原子性操作**​：使用 `@Transaction` 确保多个操作要么全部成功，要么全部回滚

#### 六、适用场景
1. ​**本地数据持久化**​：用户配置、缓存数据（如新闻列表离线阅读）
2. ​**复杂查询需求**​：多表关联查询（如电商订单与商品信息）
3. ​**响应式 `UI`​：实时数据更新（如聊天消息同步）
4. ​**数据加密**​：结合 `SQLCipher` 实现敏感数据存储
