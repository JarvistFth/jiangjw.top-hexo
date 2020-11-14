---
title: sort
date: 2020-11-14 00:38:05
categories: 
    - leetcode
    - 排序
tags: [leetcode, 排序]
keywords: [排序]
---
排序
<!---more--->

## 1365. 有多少小于当前数字的数字
https://leetcode-cn.com/problems/how-many-numbers-are-smaller-than-the-current-number/

桶排序。

```C++
class Solution {
public:
    vector<int> smallerNumbersThanCurrent(vector<int>& nums) {
        vector<int>ans;
        // for(auto n:nums){
        //     int cnt = 0;
        //     for(auto j:nums){
        //         if(j < n){
        //             cnt++;
        //         }
        //     }
        //     ans.push_back(cnt);
        // }
        // return ans;

        vector<int> vec(101);

        for(auto n:nums){
            vec[n]++;
        }

        int cnt = 0;
        for(auto& v:vec){
            int temp = v;
            v = cnt;
            cnt += temp;
        }

        for(auto n:nums){
            ans.push_back(vec[n]);
        }
        return ans;
    }
};
```
## 1122. 数组的相对排序
https://leetcode-cn.com/problems/relative-sort-array/
桶排序2。

```C++
class Solution {
public:
    vector<int> relativeSortArray(vector<int>& arr1, vector<int>& arr2) {
        if(arr1.empty()){
            return {};
        }
        vector<int> bucket(1001,0);

        for(auto i:arr1){
            bucket[i]++;
        }
        int j=0;
        for(auto x:arr2){
            while(bucket[x]){
                arr1[j++] = x;
                bucket[x]--;
            }
        }

        for(int i=0;i<bucket.size();i++){
            while(bucket[i]){
                bucket[i]--;
                arr1[j++] = i;
            }
        }
        return arr1;
    }
};
```