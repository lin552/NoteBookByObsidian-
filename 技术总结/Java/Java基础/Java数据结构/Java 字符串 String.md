---
创建时间: 2025-04-09 18:18:39
作者: wangxiaoming
tags:
  - String
  - Java
---
#### 一、构造和初始化
##### 1)直接赋值
```java
String s1 = "Hello";
```
##### 2)构造方法
```java
String s1 = new String(); //空字符串
String s2 = new String("Hello1"); //通过字符串构造
String s3 = new String(new Char[]{a,b});//字符数组构造
String s4 = new String(new Byte[]{65,66});//字节数组构造
```
##### 3)字符串常量池复用机制

###### ①核心机制​
通过字符串常量池（String Pool）复用相同内容的字符串对象，减少内存占用。字面量赋值时，JVM优先检查常量池，存在则直接引用，不存在则创建并存入池中；`new String()`强制在堆中新建对象，除非调用`intern()`手动入池

###### ②字面量赋值与new String()对比
| ​**对比维度**​     | ​**字面量赋值（`String a = "asee"`）​**​ | ​**`new String("asee")`**​ |
| -------------- | --------------------------------- | -------------------------- |
| ​**存储位置**​     | 字符串常量池（方法区）<br><br>               | 堆内存（独立对象）<br><br>          |
| ​**内存效率**​     | 高（复用机制减少重复对象）<br>                 | 低（每次创建新对象）<br><br>         |
| ​**创建方式**​     | 直接赋值，触发常量池检查<br>                  | 显式构造，绕过常量池<br><br>         |
| ​**`==`比较结果**​ | 相同内容的变量指向同一地址，返回`true`<br>        | 不同地址，返回`false`<br><br>     |
| ​**适用场景**​     | 固定字符串、高频复用场景（如配置项、枚举值）<br><br>    | 动态生成内容、需对象隔离的敏感数据<br>      |
| ​**线程安全性**​    | 天然安全（不可变对象）<br><br>               | 安全但冗余（对象独立）                |
###### ③其他细节
1. ​**常量池优化**​：
    - 字面量拼接（如`"a" + "b"`）会被编译器优化为`"ab"`，直接复用常量池对象
    - 含变量的拼接（如`String s = a + b`）会生成`StringBuilder`临时对象，最终结果在堆中
2. ​**`intern()`方法**​：  
    强制将堆中字符串引用注册到常量池，后续相同字面量可复用该引用（JDK 7+ 后常量池移至堆内存）

#### 二、字符串比较与判断
##### 1)内容比较
```java
String s1 = new String("ABC");
String s2 = new String("abc");

boolean isEqual = s1.equals(s2); //区分大小写
boolean siEqualsIgnore = s1.equalsIgnoreCase(s2); //不区分大小写
```
##### 2)字典序比较
```java
String s1 = new String("3");
String s2 = new String("1");

int result = s1.compareTo(s2);//返回差值 2 （区分大小写 字符就是返回int的差值）
int resultIgnore = s1.compareToIgnoreCase(s2); //忽略大小写
```
##### 3)前缀/后缀判断
```java
String s1 = new String("HELLO");
boolean starts = s1.startsWith("HE"); //前缀判断
boolean ends = s1.endsWith("O");
```

#### 三、字符串查找与截取
##### 1）子符/子串定位
```java
String s1 = new String("HELLO");
int lastIndex = s1.lastIndexOf("L"); //最后一次出现的位置
int indexFrom = s1.indexOf("E",0); //从指定位置开始查找
```
##### 2）截取子串
```java
String s1 = new String("HELLO");
String sub1 = s1.substring(1); //从指定位置到最后
String sub2 = s1.substring(2,4);//指定开始结束位置
```

##### 3）长度与空判断
```java
String s1 = new String("HELLO");
boolean isEmpty = s1.isEmpty(); //判断是否为空
int len = s1.length(); //字符串长度
```

#### 四、字符串转换与操作
##### 1）大小写转换
```java
String s1 = new String("HELLO");
String lower = s1.toLowerCase(); //转换为小写
String s2 = new String("hello");
String upper = s2.toUpperCase(); //转换为大写
```

##### 2）拼接与替换
```java
String s1 = new String("HELLO");
String concatStr = s1.concat(" WORLD"); //拼接字符串
String replaceStr = s1.replace('L','l'); //替换字符
String regexReplace = s1.replaceAll("[aeiou]","*"); //正则替换
```

##### 3）去空格与格式化
```java
String s1 = new String("HE LL O")
String trimmed = s1.trim(); //去除两端空白
Sring formatted = String.format("%s-%d","Java",8); //格式化字符串
```

#### 五、字符串分割与转换
##### 1)分割为数组
```java
String[] parts = "a,b,c".split(","); //按逗号分割
String[] regexSplit = "1|2".split("\\|"); //正则需转义
```
##### 2)与其他类型互转
```java
char[] charArr = s1.toCharArray(); //转换为字符数组
byte[] byteArr = s1.toBytes(); //转换为字节数组
String numStr = String.valueOf(100); //基本类型转字符串
```
#### 六、其他高频方法
##### 1)正则匹配
```java
boolean matches = s1.matches(".*llo.");
```
##### 2)内容包含检查
```java
boolean contains = s1.contanis("ell"); 
```

#### 六、优化建议
##### 1）不可变性
String 对象不可修改，每次操作会生成新对象，频繁修改建议用 `StringBuilder`
```java
StringBuilder sb = new StringBuilder();
sb.append("a").append("b"); //更高效用于替换 a+b
```
##### 2）StringBuilder 和 StringBuffer 

