---
创建时间: 2025-03-14 17:30
作者: wangxiaoming
tags:
  - Java
  - Basci-Types
  - Syntax
---
##### 一、基础类型对比

| 类型      | 占用大小 | 值范围                       | 包装类       | 默认值      | 特点                    |
| ------- | ---- | ------------------------- | --------- | -------- | --------------------- |
| byte    | 1字节  | -128 ~ 127                | Byte      | 0        | 适用于小范围整数（如文件流）        |
| short   | 2字节  | -32768 ~ 32767            | Short     | 0        | 兼容旧代码场景               |
| int     | 4字节  | -2^31 ~ 2^31-1（约-21亿~21亿） | Integer   | 0        | 默认整型，日常整数运算首选         |
| long    | 8字节  | -2^63 ~ 2^63-1            | Long      | 0L       | 用于超大数据（如时间戳）          |
| float   | 4字节  | ±1.4E-45 ~ ±3.4E+38       | Float     | 0.0f     | 单精度浮点数，精度约6-7位        |
| double  | 8字节  | ±4.9E-324 ~ ±1.8E+308     | Double    | 0.0d     | 双精度浮点数，科学计算默认类型       |
| char    | 2字节  | 0 ~ 65535（Unicode字符）      | Character | '\u0000' | 存储单个字符，支持中文等Unicode字符 |
| boolean | 1位   | true/false                | Boolean   | false    | 仅逻辑值，不可与整型混用          |
##### 二、代码示例

```java
//8种基础类型使用
byte myByte = 100; //byte类型
short myShort = 1000; //short类型
int age = 25; //int类型
long myLong = 121323L; //long类型
float myFloat = 3.14f //float类型
double pi = 3.141592635; //double类型
char myChar = 'A'; //char类型
boolean myBoolean = true; //boolean类型

//装箱操作实例
Integer a = 100; //int类型自动装箱
Integer b = Integer.valueOf(200); //int类型手动装箱

Long c = 232425L; //long类型自动装箱
Long d = Long.valueOf(2352523L); //long类型手动装箱

Double e = 3.1415926; //double类型自动装箱
Double f = Double.valueOf(3.232525); //double类型手动装箱

Character g = 'A'; //char类型自动装箱
Character h = Character.valueOf('B'); //char类型手动装箱

Boolean i = true; //boolean类型自动装箱
Boolean j = Boolean.valueOf("false"); //boolean类型手动装箱

Byte k = 100; //byte类型自动装箱
Byte l = Byte.valueOf((byte)200); //byte类型手动装箱

Short m = 30000; //short类型自动装箱
Short n = Short.valueOf((short)40000); //short类型手动装箱

Float o = 3.1344f;
Float f1 = Float.valueOf("1.618f"); // 直接解析字符串
Float f2 = Float.valueOf(1.618f);   // 需先拆箱为float，再装箱为Float
 ```
##### 三、补充

- **优先使用自动装箱:** 通过使用自动装箱也可以避免频繁创建包装类型
- **使用类型注意：** long类型使用大小写l、float类型使用大小写f