---
创建时间: 2025-04-21 10:48:45
作者: wangxiaoming
tags:
  - JSON
---
#### 一、`JSON`概述
`JSON（JavaScript Object Notation`是一种轻量级的数据交换格式，以键值对（`key-value pairs`）形式组织数据，独立于编程语言，广泛应用于网络传输、配置文件、API 交互等场景
- ​**核心特点**​：
    - 易读性强：文本格式接近自然语言，支持嵌套对象和数组
    - 高效解析：机器解析速度快，适合高频数据传输
    - 跨平台兼容：几乎所有编程语言均提供解析库（如 Java 的 `Gson`、Python 的 `json` 模块）

#### 二、`JSON`语法结构
1. ​**基本数据类型**​：
    - ​**字符串**​：双引号包裹（如 `"name": "John"`）
    - ​**数值**​：整数或浮点数（如 `"age": 30`）
    - ​**布尔值**​：`true` 或 `false`
    - ​**空值**​：`null`
2. ​**复杂结构**​：
    - ​**对象（Object）​**​：用 `{}` 包裹，键值对以逗号分隔（如 `{"name": "John", "age": 30}`）
    - ​**数组（Array）​**​：用 `[]` 包裹，元素以逗号分隔（如 `["apple", "banana"]`）
3. ​**嵌套示例**​：
```json
{
  "user": {
    "name": "John",
    "address": {"city": "New York"}
  },
  "tags": ["tech", "json"]
}
```

#### 三、`JSON`的用法与解析
1. ​**Java 中的解析库**​：
    - ​`Jackson`：高性能，支持复杂数据绑定（如嵌套对象）
```java
ObjectMapper mapper = new ObjectMapper();
Person person = mapper.readValue(jsonStr, Person.class);
```
    - ​`Gson`​：Google 开发，简单易用
```java
Gson gson = new Gson();
Person person = gson.fromJson(jsonStr, Person.class);
```
    - ​`Fastjson`：阿里巴巴开源，解析速度极快

#### 四、适用场景
1. ​**Web 开发**​：前后端数据交互（如 `RESTful API` 响应）
2. `​NoSQL` 数据库​：`MongoDB`、`CouchDB` 使用 `JSON` 存储文档数据
3. ​**配置文件**​：简化配置格式（如 VS Code 的 `settings.json`）
4. ​**日志与监控**​：记录结构化日志，便于后续分析

#### 五、注意事项与优化
1. ​**安全性**​：避免解析不可信来源的 `JSON`（可能包含恶意代码）
2. ​**兼容性**​：确保 `JSON` 键名唯一，避免重复导致解析异常
3. ​**性能优化**​：
    - ​**缓存解析结果**​：避免重复解析高频数据
    - ​**选择高效库**​：如 Java 中优先使用 `Fastjson` 或 `Jackson`
4. ​**数据验证**​：使用 `JSON Schema` 校验数据结构（如字段类型、必填项）

#### 六、面试高频考点
1. ​`JSON` vs `XML`​：`JSON` 更轻量、易解析，但 `XML` 支持注释和复杂类型验证
2. ​**解析方法对比**​：解释 `JSON.parse()` 与 `eval()` 的区别（后者可能执行恶意代码）
3. ​**泛型擦除问题**​：Java 中如何通过 `TypeToken` 解析泛型集合（如 `List<Person>`）