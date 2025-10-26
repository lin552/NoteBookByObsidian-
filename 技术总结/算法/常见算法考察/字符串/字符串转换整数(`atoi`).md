---
创建时间: 2025-06-24 13:15:39
作者: wangxiaoming
tags:
  - 字符串
---
#### 一、题目

**难度**​：中等  
​**考点**​：字符串处理、边界条件  
​**题目描述**​：  
实现一个 `atoi` 函数，使其能将字符串转换为整数。需处理前导空格、正负号、溢出等。  
​**示例**​：  
输入: `"+100"` → 输出: `100`  
输入: `" -42"` → 输出: `-42`

#### 二、解题思路
1. ​**过滤无效字符**​：跳过前导空格，处理正负号。
2. ​**转换数字**​：逐字符转换，遇到非数字停止。
3. ​**处理溢出**​：若结果超过 `Integer` 范围，返回边界值。

#### 三、代码
```java
public int myAtoi(String s){
   int index = 0,sign = 1,total = 0;
   //去除前导空格
   while(index < s.length() && s.charAt(index) == ' ') index++;
   //处理符号
   if(index < s.length() && (s.charAt(index) == '+' || s.charAt(index) == '-')){
      sign = s.charAt(index) == '+' ? 1 : -1;
      index++;
   }
   //转换数字
   while(index < s.length() && Character.isDigit(s.charAt(index))){
      int digti = s.charAt(index) - '0';
      //溢出检查
      if(Integer.MAX_VALUE / 10 < total || (Integer.MAX_VALUE / 10 == total && Integer.MAX_VALUE % 10 < digit)){
         return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
      }
      total = total * 10 + digit;
      index++;
   }
   return total * sign;
}
```