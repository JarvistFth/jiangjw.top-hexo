---
title: 前缀和
date: 2021-06-03 15:50:46
tags: [leetcode,前缀和]
categories: 
- leetcode
- 前缀和
---

前缀和相关题目

<!---more--->

## 523. 连续的子数组和
https://leetcode-cn.com/problems/continuous-subarray-sum/

```C++
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        if(n < 2){
            return false;
        }

        //当 prefixSums[q]−prefixSums[p] 为 k的倍数时，prefixSums[p] 和 prefixSums[q] 除以 k 的余数相同。
        // 因此只需要计算每个下标对应的前缀和除以 k的余数即可，使用哈希表存储每个余数第一次出现的下标。
        unordered_map<int,int> dict; //前缀和对k的余数--idx
        dict[0] = -1; //空的子数组，前缀和为0，下标规定为-1
        int remain = 0;
        for(int i=0; i<n; i++){
            remain = (remain + nums[i]) % k;
            if(dict.count(remain)){
                int prevIdx = dict[remain];
                if(i - prevIdx >= 2){
                    return true;
                }
            }else{
                dict[remain] = i;
            }
        }

        return false;
    }
};
```


## 525. 连续数组
https://leetcode-cn.com/problems/contiguous-array/

相同数量的0和1意味着和为0；所以遇到0我们让前缀和sum减1；遇见1就加1。

用一个哈希表记录当前的和的index；如果哈希表里面有某一个和的index，说明这时候是由某一对数量相同的0和1；他们的长度是当前坐标减去记录的index，也就是 i - dict[sum]。

```C++
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }

        unordered_map<int,int> dict;
        dict[0] = -1;
        int sum = 0;
        int ans = 0;
        for(int i=0; i<nums.size();i++){
            sum += (nums[i] == 0?-1:1);
            if(dict.count(sum)){
                ans = max(ans,i - dict[sum]);
            }else{
                dict[sum] = i;
            }
        }
        return ans;
    }
};
```