---
创建时间: 2025-04-09 12:25:56
作者: wangxiaoming
tags:
  - Array
  - Java
---

Array数据结构特性可查看 [[常见数据结构#1)数组（Array） Java 数组Array]]
#### 一、Java中数组使用
##### 1）声明和赋值 
###### 声明
```
int[] arr; //推荐写法
int arr[]; //不推荐，仅为兼容旧代码
```
###### 静态初始化
```
int[] arr = {1,2,3,4}; //直接赋值
```

###### 动态初始化
```
int[] arr = new int[5]; //指定长度
```

##### 2）访问和遍历
###### 通过下标访问
```
int first = arr[0]; //访问头部
int end = arr[arr.length];//访问尾部
```

###### for循环遍历
```
//for循环变理论
for(i = 1;i < arr.length;i++){
   System.out.println(arr[i]);
}

//增强for循环
for(int num : arr){
   System.out.println(num);
}
```

##### 3）其他工具
###### Arrays.copyOf() 

参数说明：
1.original 源数据  
2 .newLength 新数组长度
newLength 是几个就copy几个，可以为0，可以小于源数组

```
// 全Copy
int[] newArr = Arrays.copyOf(arr,arr.length);
```

###### System.arraycopy()

> 参数说明：
> * 1.src – 源数组.  
> * 2.srcPos – 源数组开始位置.  
> * 3.dest – 目标数组.  
> * 4.destPos – 目标数组开始位置.  
> * 5.length – copy的长度

> Tip: length 、  
> * 1)长度不能大于目标dest的长度,不然copy过去放不下 报 ArrayIndexOutOfBoundsException  
> * 2)src的长度不能小于length不然容易不够copy 报 ArrayIndexOutOfBoundsException  
> * 所以 length 范围 ( src.length => length <= dest.length )

```
//从目标src的0位置开始复制length个元素到dest的0位置
System.arraycopy(src,0,dest,0,dest.length);
```

###### Arrays常用方法

|方法|适用场景|特点说明|
|---|---|---|
|`Arrays.fill()`|初始化数组默认值或重置数组内容|高效，支持范围填充|
|`Arrays.copyOf()`|快速复制数组并调整长度|底层调用 `System.arraycopy`|
|`System.arraycopy`|高性能数组复制（需手动处理索引和长度）|原生方法，速度最快|
|`Arrays.sort()`|对数组进行排序|支持自定义比较器|

###### 总结
 * 深拷贝还是浅拷贝  ？
 
 * 一维数组：  
 * 基本数据类型：都是深拷贝，复制的是指  
 * 对象数组：浅拷贝，仅复制对象引用  
 
 * 二维数组：  
 * 任何方法均为浅拷贝，只复制外层数据的引用，内存数组与原数组共享  

