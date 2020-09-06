---
title: leetcode-贪心算法系列
date: 2020-09-06 22:17:16
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---
局部最优解就是全局最优解，选择的时候，直观就行。
<!---more--->
# 1282. 用户分组
https://leetcode-cn.com/problems/group-the-people-given-the-group-size-they-belong-to/

就用map每个group容量记录一个用户id的vector。如果map记录的该容量已满（记录的vector-size == groupSize(i)），就将记录的vector放入ans，然后清空该容量对应的vector。

```C++
class Solution {
public:
    unordered_map<int,vector<int>> map;
    vector<vector<int>> groupThePeople(vector<int>& groupSizes) {
        vector<vector<int>> ans;
        vector<int> group;
        for(int i=0;i<groupSizes.size();i++){
            map[groupSizes[i]].push_back(i);
            if(map[groupSizes[i]].size() == groupSizes[i]){
                ans.push_back(map[groupSizes[i]]);
                map[groupSizes[i]].clear();
            }
        }
        return ans;

    }
};
```

# 1221. 分割平衡字符串
https://leetcode-cn.com/problems/split-a-string-in-balanced-strings/

每次都取最短的LR字符串就可以了。比如有一个L，就要找一个R；不要多选。

```C++
class Solution {
public:
    int balancedStringSplit(string s) {
        int left = 0, right = 0;
        int ans = 0;
        for(auto c:s){
            if(c == 'L'){
                left++;
            }
            if(c == 'R'){
                right++;
            }
            if(left == right){
                ans++;
            }
        }
        return ans;
    }
};
```