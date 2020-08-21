---
title: leetcode-hashtable
date: 2020-08-04 18:44:54
categories: 
- leetcode
- 哈希表
tags: [leetcode ,哈希表]
keywords: [leetcode,哈希表]
---
leetcode，hashtable的应用。对应数据结构为unordered_set或者unordered_map。

set和map底层是红黑树，但是一些应用比较雷同，所以也放在这里吧。
<!---more--->

## 349. 两个数组的交集
https://leetcode-cn.com/problems/intersection-of-two-arrays/

这里只返回一个相同的元素，所以直接用set就可以了。将vector里面的item全部放入set，然后遍历vector2，如果set里面是有的就返回。

```C++
class Solution {
public:
    vector<int> ans;
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int> set1,set2;
        for(auto n:nums1){
            set1.insert(n);
        }

        for(auto n:nums2){
            if(set1.count(n) > 0){
                set2.insert(n);
            }
        }
        for(auto s:set2){
            ans.push_back(s);
        }
        return ans;


    }
};
```

## 350. 两个数组的交集 II
https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/

这个和上面的题不同之处在于它返回的是所有相同的元素，所以只用set是不行的，因为set只保存一个实例。这里可以用unordered_map，记录数字出现的对应次数；在vector2每遇到一次就将map里面的次数-1；如果次数为0就不返回。

```C++
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        vector<int>ans;
        unordered_map<int,int> m;//(val,count)
        for(auto n:nums1){
            m[n]++;
        }
        for(auto n:nums2){
            if(m[n] > 0){
                ans.push_back(n);
                m[n]--;
            }
        }
        return ans;
    }
};
```