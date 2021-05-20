---
title: 动态规划3-迷宫类二维dp
date: 2021-05-15 15:50:46
tags: [leetcode,动态规划]
categories: 
- leetcode
- 动态规划
---
迷宫类的二维dp。
<!---more--->

## 62. 不同路径
https://leetcode-cn.com/problems/unique-paths/

每次能从左边或者上边往目的地走，设dp[i][j]为走到i，j处不同路径的和；所以dp[i][j] = dp[i-1][j] + dp[i][j-1]；dp[0][0] = 1.

```C++
class Solution {
public:
    int ans = 0;
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m,vector<int>(n,1));
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};
```

## 63. 不同路径 II
https://leetcode-cn.com/problems/unique-paths-ii/

和上一题类似，但是多了障碍物，障碍物那一点走不了，所以
if grid[i][j] == 1{
    dp[i][j] = 0
}

```C++
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        vector<vector<int>> dp(m,vector<int>(n));
        
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(obstacleGrid[i][j] == 1){
                    dp[i][j] = 0;
                }else{
                    if(i == 0 && j == 0){
                        dp[i][j] = 1;
                    }else if(i == 0){
                        dp[i][j] = dp[i][j-1];
                    }else if(j == 0){
                        dp[i][j] = dp[i-1][j];
                    }else{
                        dp[i][j] = dp[i-1][j] + dp[i][j-1];
                    }
                }
                
            }
        }

        return dp[m-1][n-1];
    }
};
```

## 64. 最小路径和
https://leetcode-cn.com/problems/minimum-path-sum/

每一个可以从左或者从上面走下来，所以dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j]（i>=1,j>=1）；

basecase:dp[0][j]和dp[i][0]应该为对应的grid[0][j]或grid[i][0]的和。

```C++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> dp(m,vector<int>(n));
        dp[0][0] = grid[0][0];
        for(int j=1; j<n; j++){
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }
        for(int i=1; i<m; i++){
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }

        for(int i=1; i<m; i++){
            for(int j=1; j<n; j++){
                dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j];
            }
        }

        return dp[m-1][n-1];
    }
};
```

## 221. 最大正方形
https://leetcode-cn.com/problems/maximal-square/

这道题其实是考二维矩阵里面的最大连续边长。定义dp[i][j]为坐标（i-1，j-1）处最大正方形的边长。

首先，dp[i][j]当且仅当matrix[i-1][j-1]是1时才需要计算，才有意义。如果是0，就没有办法构成正方形了，值就是0。

如果mat[i-1][j-1]是1，构成正方形由左上，左，上三个位置共同构成（因为我们是从0到尽头遍历的）。所以dp[i][j] = 1 + min({dp[i-1][j],dp[i][j-1],dp[i-1][j-1]})。

因为如果它的上方，左方和左上方为右下角的正方形的大小不一样，合起来就会缺了某个角落，这时候只能取那三个正方形中最小的正方形的边长加1了。

```C++
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {


        int m = matrix.size(), n = matrix[0].size();
        int edge = 0;
        vector<vector<int>> dp(m+1,vector<int>(n+1));
        for(int i=1; i<=m; i++){
            for(int j=1; j<=n; j++){
                if(matrix[i-1][j-1] == '1'){
                    dp[i][j] = 1 + min({dp[i-1][j],dp[i][j-1],dp[i-1][j-1]});
                    edge = max(edge,dp[i][j]);
                }
            }
        }
        return edge*edge;


    }
};
```



