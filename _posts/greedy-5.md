---
title: greedy-5
date: 2020-09-13 21:43:47
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---
贪心问题，按照通过率排序，第五、六页题目。

<!---more--->

## 135. 分发糖果
https://leetcode-cn.com/problems/candy/

分别从左右遍历一次数组，保证其相邻的分数高者获得较多的糖果。

正向遍历一次，找到所有右边分数高的给多一颗糖果；

反向遍历一次，找到所有左边分数高的，如果这时候他的糖果数量还是小于等于右边的孩子，就再给他多一颗。

```C++
class Solution {
public:
    int candy(vector<int>& ratings) {
        if(ratings.empty()){
            return 0;
        }

        vector<int> diliver(ratings.size());
        diliver[0] = 1;

        for(int i=1;i<ratings.size();i++){
            if(ratings[i] > ratings[i-1]){
                diliver[i] = diliver[i-1]+1;
            }else{
                diliver[i] = 1;
            }
        }

        for(int i=ratings.size()-2; i>=0; i--){
            if(ratings[i] >ratings[i+1]){
                if(diliver[i] <= diliver[i+1]){
                    diliver[i] = diliver[i+1]+1;
                }
            }
        }

        int ans = 0;
        for(auto i:diliver){
            ans+=i;
        }
        return ans;
    }
};
```

## 435. 无重叠区间
https://leetcode-cn.com/problems/non-overlapping-intervals/

按照区间的右端点排列，然后判断当前的左端点和前一个右端点的大小；如果是大于等于前一个的，就证明没有重叠。

然后把全部集合的数量减去没有重叠的数量，就是答案。

```C++
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        if(intervals.empty()){
            return 0;
        }
        int ans = 0;

        sort(intervals.begin(),intervals.end(),[](auto& a,auto& b){
            return a[1]==b[1]?a[0]<b[0]:a[1]<b[1];
        });

        int pre = intervals[0][1];
        int match = 1;

        for(int i=1;i<intervals.size();i++){
            if(intervals[i][0] >= pre){
                pre = intervals[i][1];
                match++;
            }
        }
        return intervals.size() - match;
    }
};
```

## 55. 跳跃游戏
https://leetcode-cn.com/problems/jump-game/

对每一个可以到达的位置（i<=distance）都维护一个能达到的最大距离(max(distance,nums[i]+i)。只要最后最大距离是超出数组长度，就可以到达。

```C++
class Solution {
public:
    bool canJump(vector<int>& nums) {
        if(nums.empty()){
            return false;
        }

        int distance = nums[0];
        for(int i=1;i<nums.size()-1;i++){
            if(i <= distance){
                distance = max(distance,nums[i]+i);
            }
        }
        return distance >= nums.size()-1;
    }
};
```