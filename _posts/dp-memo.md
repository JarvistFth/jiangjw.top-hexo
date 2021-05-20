---
title: 动态规划4-记忆化搜索
date: 2021-05-16 15:50:46
tags: [leetcode,动态规划]
categories: 
- leetcode
- 动态规划
---


## 494. 目标和
https://leetcode-cn.com/problems/target-sum/

给你一个整数数组 nums 和一个整数 target 。

向数组中的每个整数前添加 '+' 或 '-' ，然后串联起所有整数，可以构造一个 表达式 ：

例如，nums = [2, 1] ，可以在 2 之前添加 '+' ，在 1 之前添加 '-' ，然后串联起来得到表达式 "+2-1" 。
返回可以通过上述方法构造的、运算结果等于 target 的不同 表达式 的数目。

---
记忆化搜索，记录每次当前位置选择以后对应的和作为key，表达式数目作为value。最后返回的就是memo[(size-1,sum)]对应的value。

为了方便做hash，用string作为key的类型，通过当前选择的坐标(start)和选择以后的sum构建string-key。

每次选择可以做加或者减。如果是加，dfs下一个的sum就是sum - nums[start]；如果是减，dfs下一个sum就是sum + nums[start]；

什么时候退出dfs？每次选择的index超出了nums的长度的时候。

这时候看一下sum是否为0，是0，证明这次选择可以构造成sum，return 次数1.

```C++
class Solution {
public:
    unordered_map<string,int> memo;
    //<<start,sum>,ans>
    int findTargetSumWays(vector<int>& nums, int S) {
        return dfs(nums,0,S);;
    }

    int dfs(vector<int>& nums,int start, long sum){
        if(start >= nums.size()){
            if(sum == 0){
                return 1;
            }
            return 0;
        }

        string key = to_string(start) + "/" + to_string(sum);
        if(!memo.count(key)){
            memo[key] = dfs(nums,start+1,sum - nums[start]) + dfs(nums,start+1,sum + nums[start]);
            return memo[key];
        }else{
            return memo[key];
        }
        
    }
};
```