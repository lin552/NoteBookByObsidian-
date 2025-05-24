---
创建时间: 2025-04-15 11:00:17
作者: wangxiaoming
tags:
  - SQLite
  - SQL
---
#### 一、基础表结构操作
##### 1）创建表
```sql
CREATE TABLE IF NOT EXISTS students (
    student_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(50) NOT NULL,
    gender TEXT CHECK(gender IN ('男', '女')) DEFAULT '男',
    birth_date DATE,
    major VARCHAR(100)
);
```
##### 2）插入数据
```sql
INSERT INTO students (name, gender, birth_date, major)  
VALUES  
('张三','男','2003-05-12','计算机科学'),  
('李四','男','2002-11-08','数据科学'),  
('王芳','女','2008-06-23','物联网'),  
('小美','女','2004-05-20','程序鼓励'),  
('小强','男','2016-03-24','游戏鉴赏'),  
('小成','男','1996-02-15','网络工程'),  
('小天','男','1998-07-14','野区王者');
```
#### 二、数据查询操作
##### 1）基础查询
```sql
-- 查询表里所有
SELECT * FROM students;

-- 查询指定列并别名显式  
SELECT name AS 姓名, major AS 专业 FROM students;

-- 过滤空值出生日期  
SELECT * FROM students WHERE birth_date IS NOT NULL;
```
##### 2）条件查询
```sql
--范围查询:2003年后出生的学生  
SELECT * FROM students  
WHERE birth_date > '2003-01-01';

-- 模糊查询：专业包含"科学"  
SELECT * FROM students  
WHERE major LIKE  '%科学%';

-- 组合条件:女性且专业为物联网  
SELECT * FROM students  
WHERE gender = '女' AND major = '物联网';
```
#### 三、进阶操作
##### 1）创建关联课程表
```sql
-- 课程表  
CREATE TABLE courses  
(  
    course_id   INT PRIMARY KEY,  
    course_name VARCHAR(100),  
    credit      INT CHECK ( credit BETWEEN 1 AND 5)  
);  
  
--学生选课关联表  
CREATE TABLE enrollments(  
    student_id INT,  
    course_id INT,  
    enrollment_date DATE,  
    FOREIGN KEY (student_id) REFERENCES students(student_id),  
    FOREIGN KEY (course_id) REFERENCES courses(course_id)  
);
```

#####  2）多表关联查询
```sql
-- 查询张三的选课记录  
SELECT s.name,c.course_name,e.enrollment_date  
FROM students s  
JOIN enrollments e ON s.student_id = e.student_id  
JOIN courses c ON e.course_id = c.course_id  
WHERE s.name = '张三';
```

##### 3）聚合统计
```sql
-- 统计各专业学生人数  
SELECT students.major,COUNT(*) AS total  
FROM students  
GROUP BY major  
ORDER BY total DESC;

--计算课程平均学分  
SELECT AVG(courses.credit) AS avg_credit  
FROM courses

```
#### 四、数据维护操作
##### 1）更新数据
```sql
--调整李四的专业  
UPDATE students  
SET major = '大数据分析'  
WHERE name = '李四';

--批量增加学分（所有课程+0.5）  
UPDATE courses  
SET credit = credit + 0.5  
WHERE credit < 4;
```

##### 2）删除操作
```sql
-- 删除无选课记录的学生  
DELETE FROM students  
WHERE student_id NOT IN (  
    SELECT DISTINCT student_id FROM enrollments  
);
```