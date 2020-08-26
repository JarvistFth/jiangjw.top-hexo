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
