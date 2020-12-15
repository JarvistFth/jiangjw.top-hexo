---
title: Leetcode - sort
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

## 406. 根据身高重建队列
https://leetcode-cn.com/problems/queue-reconstruction-by-height/

将身高按降序、同一身高排序按升序排序。

这样的好处是按照按照排序后的顺序进行头插法，立刻就满足了要求。（否则按照位置降序，第一个进行头插法的就没法插入了）

然后按照位置的序号，进行头插法，插入到数组中。

```C++
class Solution {
public:
    
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(),people.end(),[](auto &a, auto &b){
            return a[0] == b[0]? a[1]<b[1]:a[0]>b[0];
        });
        vector<vector<int>> ans;
        for(auto &p:people){
            ans.insert(ans.begin()+p[1],p);
        }
        return ans;
    }
};
```