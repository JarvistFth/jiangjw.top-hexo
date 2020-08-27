---
title: 动态规划-1
date: 2020-08-26 23:50:46
tags: [leetcode,动态规划]
categories: 
- leetcode
- 动态规划
---
LeetCode动态规划题目，先从简单的做起。。。。。


<!---more--->

# 53. 最大子序和
求最大子序和，所以状态f[i]表示，选了第i个数字后的子序和。

当我们不选i的时候，题目要求是连续的子序和；所以f[i] = nums[i]；

当我们选i的时候，f[i]显然是f[i-1]+nums[i]。

所以状态转移方程就是f[i] = max(num[i],f[i-1]+nums[i]);

初始状态就是f[0]，选nums[0]，所以发f[0] = nums[0]。

因为循环从1开始，所以ans去nums[0]。

答案就是max(ans,f[i])；

```C++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        vector<int> dp(nums.size()+2);
        int ans = nums[0];
        dp[0] = nums[0];
        for(int i=1;i<nums.size();i++){
            dp[i] = max(nums[i],dp[i-1]+nums[i]);
            ans = max(ans,dp[i]);
        }

        return ans;
    }
};
```

# 300. 最长上升子序列
https://leetcode-cn.com/problems/longest-increasing-subsequence/

f[i]表示选了nums[i]的时候的上升子序列的最长长度；

如果我们选nums[i]，要保证他比之前所有的nums[j](j< i)要大，这样选了之后的最长长度就是当前的长度和之前所有上升子序列长度+1（dp[j]+1)较大的那一方。

最后，我们得到的f[i]是以选了nums[i]的上升子序列的最长长度，要得到所有元素的最长长度，还要遍历一遍获得其最大值。

```C++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if(nums.size() == 0){
            return 0;
        }
        vector<int> dp(nums.size()+2,1);

        for(int i=0;i<nums.size();i++){
            for(int j=0;j<i;j++){
                if(nums[i] > nums[j]){
                    dp[i] = max(dp[i],dp[j]+1);
                }
            }
        }

        int ans = 0;
        for(auto n:dp){
            ans = max(ans,n);
        }

        return ans;

    }
};
````
# 983. 最低票价
https://leetcode-cn.com/problems/minimum-cost-for-tickets/

f[i]对应第i天的最低票价。

对于第i天，如果这一天不是出行的日子，那么花费还是f[i-1]；

如果是出行的日子，那么我们有三种选择；分别是买1天、7天和30天的票。

对于这三种选择，我们取各自花费的最小值。

因为我们f[i]存的就是第i天花费的最小值，而某票价是确定的。所以各自的花费最小值，应该对应f[i-1]/f[i-7]/f[i-30]+对应费用。

最后我们返回i是最后一天的f[i]就可以了。
```C++
class Solution {
public:
    int mincostTickets(vector<int>& days, vector<int>& cost) {
        vector<int> dp(days.back()+2,0);

        for(auto d:days){
            dp[d] = INT_MAX;
        }

        for(int i=1;i<=days.back();i++){
            if(dp[i] == 0){
                dp[i] = dp[i-1];
            }else{
                dp[i] = min({dp[i-1] + cost[0],dp[max(0,i-7)]+cost[1],dp[max(0,i-30)]+cost[2]});
            }
        }
        return dp[days.back()];
    }
};
```

# 70. 爬楼梯
https://leetcode-cn.com/problems/climbing-stairs/

f[i]为爬i层楼梯的总数；我们的选择可以分为先爬i-1层楼梯再爬1层；或者先爬i-2层再爬2层。

显然先爬i-1层楼梯总数为f[i-1]，先爬i-2层总数为f[i-2]。

所以f[i] = f[i-1] + f[i-2]。

```C++
class Solution {
public:
    int times = 0;
    int climbStairs(int n) {
        
        if(n<=2){
            return n;
        }
        int *dp = new int [n+1];
        dp[1] = 1;
        dp[2] = 2;
        for(int i=3;i<=n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
};
```

# 剑指 Offer 47. 礼物的最大价值

f[i,j]表示在[i,j]处能拿到礼物的最大价值；

那么显然f[i][j] = max(f[i-1][j]+grid[i][j],f[i][j-1]+grid[i][j])。

考虑baseline，f[0][j]表示在[0,j]处能拿到礼物的最大值，所以是为第j列拿到的价值和；
f[i][0]是[i,0]处能拿到礼物的最大值，所以是第i行的价值和。

优化：观察到dp[i][j]其实只与j有关，所以可以缩小一个维度。dp[j] = max(dp[j-1],dp[j]) + grid[i][j]。这里i,j要从0开始，因为grid[0][0]都要处理。

为了方便处理边界条件，可以考虑i,j从1开始，创建的dp[j]是grid.size()+1. 这样我们dp的下标是从1开始，返回的时候返回grid.size下标处即可。

```C++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        // vector<vector<int>> dp(grid.size(),vector<int>(grid[0].size()));
        vector<int>dp(grid[0].size()+1);



        // int sum = 0;
        // for(int i=0;i<grid.size();i++){
        //     sum+=grid[i][0];
        //     dp[i][0] = sum;
        // }
        // sum = 0;
        // for(int j=0;j<grid[0].size();j++){
        //     sum+=grid[0][j];
        //     dp[0][j] = sum;
        // }

        for(int i=1;i<=grid.size();i++){
            for(int j=1;j<=grid[0].size();j++){
                //dp[i][j] = max(dp[i-1][j],dp[i][j-1]) + grid[i][j]
                dp[j] = max(dp[j],dp[j-1]) + grid[i-1][j-1];
            }
        }
        return dp[grid[0].size()];
    }
};
```
