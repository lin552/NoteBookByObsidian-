---
创建时间: "2025-06-13 20:43:33"
作者: wangxiaoming
tags:
---
#### 一、连接查询（JOIN）：最常用的多表关联方式

连接查询通过表之间的**关联字段**​（如外键）将多个表的数据关联起来，根据关联规则的不同分为以下类型：
##### 1.内连接（INNER JOIN）：仅返回匹配的行
内连接是最常用的连接方式，仅返回两个表中**关联字段匹配**的行。若某行在关联表中无匹配项，则不会出现在结果中。
```sql
SELECT [列名列表]
FROM 表1
INNER JOIN 表2 
ON 表1.关联字段 = 表2.关联字段
[WHERE 过滤条件];
```
**示例**​：  
假设有 `students`（学生表）和 `scores`（成绩表），需查询“有成绩的学生姓名和分数”：
```sql
-- 表结构
-- students: id（学号）, name（姓名）
-- scores: sid（学号）, subject（科目）, score（分数）

SELECT s.name, sc.subject, sc.score
FROM students s
INNER JOIN scores sc 
ON s.id = sc.sid; -- 关联条件：学生表的 id = 成绩表的 sid
```
**结果**​：仅包含有成绩的学生的记录（无成绩的学生不会出现在结果中）。

##### 2.左外连接（LEFT JOIN / LEFT OUTER JOIN）：返回左表所有行，右表匹配或 NULL
左外连接以**左表（第一个表）​**为基准，返回左表的所有行，右表中匹配的行；若右表无匹配，则右表字段用 `NULL` 填充。
```sql
SELECT [列名列表]
FROM 表1
LEFT JOIN 表2 
ON 表1.关联字段 = 表2.关联字段
[WHERE 过滤条件];
```
示例：
查询“所有学生（包括无成绩的）的姓名和对应科目成绩”：
```sql
SELECT s.name, sc.subject, sc.score
FROM students s
LEFT JOIN scores sc 
ON s.id = sc.sid; -- 左表是 students，右表是 scores
```
**结果**​：所有学生的记录都会出现，无成绩的学生的 `subject` 和 `score` 为 `NULL`。

##### 3.右外连接（RIGHT JOIN / RIGHT OUTER JOIN）：返回右表所有行，左表匹配或 NULL
右外连接与左外连接相反，以**右表（第二个表）​**为基准，返回右表的所有行，左表中匹配的行；若左表无匹配，则左表字段用 `NULL` 填充。
```sql
SELECT [列名列表]
FROM 表1
RIGHT JOIN 表2 
ON 表1.关联字段 = 表2.关联字段;
```
示例：
查询“所有有成绩的科目对应的学生姓名（包括无学生的科目记录）”：
```sql
SELECT s.name, sc.subject, sc.score
FROM students s
RIGHT JOIN scores sc 
ON s.id = sc.sid; -- 右表是 scores，左表是 students
```
结果：所有有成绩的科目记录都会出现，无对应学生的科目 `name` 为 `NULL`。

##### 4.全外连接（FULL JOIN / FULL OUTER JOIN）：返回左右表所有行，无匹配则 NULL
全外连接返回左表和右表的**所有行**，若某行在另一表中无匹配，则对应字段用 `NULL` 填充。  
​**注意**​：MySQL 不支持 `FULL JOIN`，需用 `LEFT JOIN UNION RIGHT JOIN` 模拟。
```sql
SELECT [列名列表]
FROM 表1
FULL JOIN 表2 
ON 表1.关联字段 = 表2.关联字段;
```
**示例（模拟 MySQL 全外连接）​**​：
```sql
-- 等价于 FULL JOIN
(SELECT s.name, sc.subject, sc.score
FROM students s
LEFT JOIN scores sc ON s.id = sc.sid)
UNION
(SELECT s.name, sc.subject, sc.score
FROM students s
RIGHT JOIN scores sc ON s.id = sc.sid);
```

##### 5.交叉连接（CROSS JOIN）：笛卡尔积（慎用！）
交叉连接返回两个表的**所有可能组合**​（行数 = 表1行数 × 表2行数），通常需配合 `WHERE` 过滤无效数据。  
​**注意**​：无 `ON` 条件时，`CROSS JOIN` 等价于笛卡尔积，可能导致结果集爆炸。
```sql
SELECT [列名列表]
FROM 表1
CROSS JOIN 表2
[WHERE 过滤条件];
```
**示例**​：  
查询“所有学生与所有科目的组合（假设需要生成考试安排）”：
```sql
SELECT s.name, sc.subject
FROM students s
CROSS JOIN subjects sc; -- subjects 是科目表
```
#### 二、子查询（`Subquery`）:嵌套查询
子查询是将一个 SQL 查询的结果作为另一个查询的输入，适用于复杂条件过滤或数据关联。常见子查询类型：
##### 1.WHERE子句中的子查询（标量子查询）
用于在 `WHERE` 条件中过滤数据，子查询返回单个值（标量）。
**示例**​：查询“分数高于平均分的学生”：
```sql
SELECT name, score
FROM scores
WHERE score > (SELECT AVG(score) FROM scores); -- 子查询返回平均分
```

##### 2.FROM子句中的子查询（派生表）
子查询作为临时表（派生表），可在 `FROM` 子句中引用，适用于多表关联后的复杂过滤。
**示例**​：查询“各科目的平均分，并筛选平均分大于 80 的科目”：
```sql
SELECT subject, avg_score
FROM (
    SELECT sc.subject, AVG(sc.score) AS avg_score
    FROM scores sc
    GROUP BY sc.subject
) AS subject_avg -- 派生表别名
WHERE avg_score > 80;
```

##### 3.SELECT 子句中的子查询（标量子查询）
子查询返回单个值，作为 `SELECT` 列的一部分。
​**示例**​：查询“学生姓名及其所在班级的平均分”（假设 `students` 表有 `class_id` 字段）：
```sql
SELECT 
    s.name,
    (SELECT AVG(sc.score) 
     FROM scores sc 
     WHERE sc.sid = s.id) AS class_avg -- 子查询返回当前学生的平均分
FROM students s;
```

#### 三、集合操作（UNION/UNION ALL）：合并结果集
`UNION` 和 `UNION ALL` 用于合并两个或多个 `SELECT` 语句的结果集，要求各查询的**列数、列类型一致**。

##### 1.UNION：去重合并
`UNION` 会自动去重，合并后的结果集中不包含重复行。
```sql
SELECT 列1, 列2 FROM 表1
UNION
SELECT 列1, 列2 FROM 表2;
```
**示例**​：查询“所有男生和女生的姓名（去重）”：
```sql
SELECT name FROM boys
UNION
SELECT name FROM girls;
```

##### 2.UNION ALL：不去重合并
`UNION ALL` 直接合并结果集，保留所有行（包括重复行），性能优于 `UNION`（无需去重）。
```sql
SELECT 列1, 列2 FROM 表1
UNION ALL
SELECT 列1, 列2 FROM 表2;
```
**示例**​：查询“所有男生和女生的姓名（包含重复）”：
```sql
SELECT name FROM boys
UNION ALL
SELECT name FROM girls;
```

#### 四、多表查询的选择依据
根据需求选择合适的多表查询方式：
- ​**需要关联匹配数据**​ → 内连接（`INNER JOIN`）。
- ​**需要保留主表所有数据（含无关联的）​**​ → 左/右外连接（`LEFT/RIGHT JOIN`）。
- ​**需要所有表数据（含无关联的）​**​ → 全外连接（`FULL JOIN`）或 `LEFT+RIGHT UNION`。
- ​**需要所有可能的组合**​ → 交叉连接（`CROSS JOIN`），但需谨慎使用。
- ​**复杂条件过滤或嵌套逻辑**​ → 子查询（`Subquery`）。
- ​**合并多个表的结果集**​ → `UNION/UNION ALL`。

#### 五、注意事项
1. ​**避免笛卡尔积**​：连接时务必指定 `ON` 条件，否则会导致结果集爆炸（尤其是大表）。
2. ​**别名简化语句**​：为表设置别名（如 `s` 代表 `students`），提高可读性。
3. ​**性能优化**​：过多连接或子查询可能影响性能，可通过索引优化关联字段，或减少不必要的列查询。
4. ​**数据库差异**​：部分语法（如 `FULL JOIN`）在不同数据库中支持不同，需根据目标数据库调整。