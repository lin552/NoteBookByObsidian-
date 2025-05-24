---
创建时间: 2025-05-20 15:48:59
作者: wangxiaoming
tags:
  - 数组排序
  - 算法
---

##### 1）题目
传送带上的包裹必须在 `days` 天内从一个港口运送到另一个港口。

传送带上的第 `i` 个包裹的重量为 `weights[i]`。每一天，我们都会按给出重量（`weights`）的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。

返回能在 `days` 天内将传送带上的所有包裹送达的船的最低运载能力。

**示例 1：**

**输入：**weights = [1,2,3,4,5,6,7,8,9,10], days = 5
**输出：**15
**解释：**
船舶最低载重 15 就能够在 5 天内送达所有包裹，如下所示：
第 1 天：1, 2, 3, 4, 5
第 2 天：6, 7
第 3 天：8
第 4 天：9
第 5 天：10

请注意，货物必须按照给定的顺序装运，因此使用载重能力为 14 的船舶并将包装分成 (2, 3, 4, 5), (1, 6, 7), (8), (9), (10) 是不允许的。 

**示例 2：**

**输入：**weights = [3,2,2,4,1,4], days = 3
**输出：**6
**解释：**
船舶最低载重 6 就能够在 3 天内送达所有包裹，如下所示：
第 1 天：3, 2
第 2 天：2, 4
第 3 天：1, 4

**示例 3：**

**输入：**weights = [1,2,3,1,1], days = 4
**输出：**3
**解释：**
第 1 天：1
第 2 天：2
第 3 天：3
第 4 天：1, 1

**提示：**

- `1 <= days <= weights.length <= 5 * 104`
- `1 <= weights[i] <= 500`

##### 2) 解题思路

求最低运载能力，即每天最少运多少，可以在days内运完
假定这个值是x,用二分查找法，在区间内一直缩减
范围是 `weights 中max -> weights 总和`区间内

##### 3）代码示例
```java
class Solution {

    public int shipWithinDays(int[] weights, int days) {

        //left为最大重量 right为总重
       int left = Arrays.stream(weights).max().getAsInt(),right = Arrays.stream(weights).sum();
       //二分查找法
       while(left < right){
          int mid = (left + right) / 2;
          int need = 1,cur = 0;
          //遍历weights
          for(int weight : weights){
            //如果总量大于mid就说明至少需要一天来运输
            //所以need是需要运输的天数
            //cur为当前这一天已经运送的包裹重量之和
            if(cur + weight > mid){
                ++need;
                cur = 0;
            }
            cur += weight;
          }
          //如果need小于days就边界左缩
          //否则边界右缩
          //直到返回最小
          if(need <= days){
            right = mid;
          } else {
            left = mid + 1;
          }
       }
       return left;
    }

}
```