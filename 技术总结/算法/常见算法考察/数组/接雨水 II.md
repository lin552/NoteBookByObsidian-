---
创建时间: 2025-06-24 13:20:04
作者: wangxiaoming
tags:
  - 数组
---
#### 一、题目

**难度**​：中等  
​**考点**​：优先队列、`BFS`、边界处理  
​**题目描述**​：  
给定一个表示非负整数的矩阵，计算按此排列的柱子，下雨之后能接多少雨水。
```
[
 [1,4,3,1,3,2],
 [3,2,1,3,2,4],
 [2,3,3,2,3,1]
]
```
#### 二、解题思路
1. ​**边界优先处理**​：将矩阵边缘的柱子加入优先队列（最小堆），记录已访问的节点。
2. ​**`BFS`遍历**​：每次取出高度最小的柱子，检查其四周的柱子，计算可接雨水量。

#### 三、代码
```java
public int trapRainWater(int[][] heightMap){
   if(heigthMap == null || heightMap.length == 0) return 0;
   int m = heightMap.length, n = heightMap[0].length;
   PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[2] - b[2]);
   boolean[][] visited = new boolean[m][n];
   //将边界加入队列
   for(int i = 0;i < m;i++){
      pq.offer(new int[]{i,0,heightMap[i][0]});
      pq.offer(new int[]{i,n-1,heightMap[i][n-1]});
      visited[0][j] = visited[m-1][j] = true;
   }
   int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};

    int res = 0;
    while (!pq.isEmpty()) {
        int[] cell = pq.poll();
        for (int[] dir : dirs) {
            int x = cell[0]+ dir[0], y = cell[1]+ dir[1];
            if (x >= 0 && x < m && y >= 0 && y < n && !visited[x][y]) {
                res += Math.max(0, cell[2] - heightMap[x][y]);
                pq.offer(new int[]{x, y, Math.max(cell[2], heightMap[x][y])});
                visited[x][y] = true;
            }
        }
    }
    return res;
}
```