---
创建时间: "2025-08-28 16:37:51"
作者: wangxiaoming
tags:
---
在 Room 中，数据库升级（版本迁移）是核心功能之一，主要通过**版本号控制**和**迁移策略**实现。以下从**升级流程**、**迁移方式**​（手动/自动）、**其他常用技巧**三个维度详细说明，并附代码示例。

#### 一、数据库升级的核心流程

Room 数据库升级的本质是**处理版本号变化**。当通过 `@Database(version = N)`增加版本号时，Room 会检测到版本差异，并触发迁移逻辑。若未提供迁移策略，Room 会直接崩溃（抛出 `IllegalStateException`）。

#### 二、手动迁移：自定义 Migration对象
最灵活的方式是手动定义迁移规则，适用于**复杂表结构变更**​（如修改列、删除表、添加索引等）。
##### 1.基础步骤
1. **增加数据库版本号**​：修改 `@Database`注解的 `version`参数（如从 1 → 2）。
2. ​**定义迁移对象**​：创建 `Migration`实例，指定旧版本（`startVersion`）和新版本（`endVersion`），并在 `migrate()`方法中编写 SQL 逻辑。
3. ​**绑定迁移策略**​：通过 `Room.databaseBuilder().addMigrations(migration)`添加迁移。
##### 2. 示例：添加新列
假设原数据库（版本 1）有表 `User`，字段为 `id`和 `name`；升级到版本 2 时需要添加 `age`列。
```kotlin
// 1. 定义迁移：从版本 1 → 2
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // 执行 SQLite ALTER TABLE 语句添加列
        database.execSQL("ALTER TABLE User ADD COLUMN age INTEGER DEFAULT 0")
    }
}

// 2. 构建数据库时绑定迁移
val db = Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java, "app-db"
)
.addMigrations(MIGRATION_1_2) // 添加迁移规则
.build()
```
##### 3.复杂迁移场景
- **删除列**​（SQLite 不支持直接删除列，需间接操作）：
需创建临时表 → 复制旧表数据（排除待删除列） → 删除旧表 → 重命名临时表。
```kotlin
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // 1. 创建临时表（不含待删除的 'age' 列）
        database.execSQL("""
            CREATE TABLE User_temp (
                id INTEGER PRIMARY KEY NOT NULL,
                name TEXT NOT NULL
            )
        """)
        // 2. 复制数据（排除 age 列）
        database.execSQL("INSERT INTO User_temp SELECT id, name FROM User")
        // 3. 删除旧表
        database.execSQL("DROP TABLE User")
        // 4. 重命名临时表为新表
        database.execSQL("ALTER TABLE User_temp RENAME TO User")
    }
}
```
- **添加新表**：直接执行 `CREATE TABLE`语句。
```kotlin
val MIGRATION_3_4 = object : Migration(3, 4) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("""
            CREATE TABLE Profile (
                userId INTEGER PRIMARY KEY,
                bio TEXT,
                FOREIGN KEY(userId) REFERENCES User(id)
            )
        """)
    }
}
```
##### 4.多版本连续迁移
若版本跳跃（如从 1 → 3），需添加所有中间版本的迁移（1→2 和 2→3）：
```kotlin
.addMigrations(MIGRATION_1_2, MIGRATION_2_3)
```

#### 三、自动迁移：简化简单变更
Room 2.4+ 引入了**自动迁移（Auto Migration）​**，可自动生成简单表结构变更的迁移代码（如添加列、添加表、删除索引等），无需手动写 SQL。
##### 1.启用自动迁移
•**无数据丢失的简单变更**​：直接使用 `autoMigrate()`。
```kotlin
val db = Room.databaseBuilder(...)
    .addMigrations(
        // 自动处理 1→2 的迁移（如添加列）
        Migration(1, 2), 
        // 或使用 emptyMigrations 声明无数据丢失的自动迁移
        emptyMigrations(2, 3) 
    )
    .build()
```
•**需指定导出 Schema**​：自动迁移依赖 `exportSchema = true`（默认开启），生成的 Schema 文件（`room/master_table`）会记录表结构变化。

##### 2.限制与注意事项
- ​**支持的变更**​：添加列（非主键）、添加表、添加/删除索引、重命名表/列（需保持类型兼容）。
- ​**不支持的变更**​：删除列、修改列类型（如 `TEXT`→ `INTEGER`）、修改主键。
- ​**混合使用**​：复杂变更（如删除列）需结合手动迁移和自动迁移。

#### 四、其他常见技巧
##### 1.预填充数据库（`Prepopulate`）
首次创建数据库时，可通过 `createFromAsset()`或 `createFromFile()`预填充初始数据（如字典、配置表）。
```kotlin
val db = Room.databaseBuilder(...)
    .createFromAsset("database/initial.db") // 从 assets 目录加载预填充数据库
    .build()
```
**注意**​：预填充数据库的版本号需与 `@Database(version)`一致，否则会触发迁移。

##### 2.数据库回调（Callback）
通过 `RoomDatabase.Callback`监听数据库的创建或打开事件，执行初始化或清理操作（如插入测试数据）。
```kotlin
val callback = object : RoomDatabase.Callback() {
    override fun onCreate(db: SupportSQLiteDatabase) {
        super.onCreate(db)
        // 数据库首次创建时执行（如插入初始用户）
        CoroutineScope(Dispatchers.IO).launch {
            db.execSQL("INSERT INTO User (id, name) VALUES (1, 'Admin')")
        }
    }

    override fun onOpen(db: SupportSQLiteDatabase) {
        super.onOpen(db)
        // 数据库每次打开时执行（如清理过期缓存）
    }
}

val db = Room.databaseBuilder(...)
    .addCallback(callback)
    .build()
```

##### 3.多进程访问
若需支持多进程（如主进程和后台服务），需在 `@Database`中设置 `multiProcess = true`，并通过 `Room.databaseBuilder().allowMainThreadQueries()`（不推荐）或在后台线程操作。
```kotlin
@Database(entities = [User::class], version = 1, multiProcess = true)
abstract class AppDatabase : RoomDatabase() { ... }
```

##### 4.导出Schema(重要)
Room 默认会生成 Schema 文件（路径：`build/generated/source/kapt/debug/.../room/master_table`），记录所有表的元数据。
- **作用**​：用于自动迁移、版本回溯、调试。
- ​**强制开启**​：若需显式声明，在 `@Database`中添加 `exportSchema = true`（默认已开启）。

#### 五、最佳实践
1. ​**版本控制**​：严格递增 `version`，避免跳跃式升级（如 1→3 需补全 1→2 的迁移）。
2. ​**优先自动迁移**​：简单变更（添加列、表）使用 `emptyMigrations`自动生成，减少代码量。
3. ​**手动迁移保复杂**​：涉及数据删除、类型修改等复杂操作时，手动编写 `Migration`并测试。
4. ​**测试迁移逻辑**​：通过单元测试（如 `AndroidJUnitRunner`）模拟升级过程，验证数据完整性。
5. ​**预填充与回调**​：合理使用预填充减少初始化代码，通过回调处理初始化或清理逻辑。