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

### 349. 两个数组的交集
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

### 350. 两个数组的交集 II
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

### 532. 数组中的K-diff数对
https://leetcode-cn.com/problems/k-diff-pairs-in-an-array/

先将数组放入哈希表；然后再哈希表中找当前的值+k在哈希表中是否存在；如果存在ans++。因为这里只返回一个数对，所以(1,3)和(3,1)是一样的，所以直接正向+k就可以。另外注意需要特判k是否为0 。如果是的话，只有出现次数大于1的数字，才能作为数对进行返回。

```C++
class Solution {
public:
    int findPairs(vector<int>& nums, int k) {
        unordered_map<int,int> map;
        int ans = 0;
        if(k<0){
            return 0;
        }

        for(auto n:nums){
            map[n]++;
        }

        if(k == 0){
            for(auto m:map){
                if(m.second > 1){
                    ans++;
                }
            }
            return ans;
        }


        for(auto m:map){
            if(map.count(m.first+k)){
                ans++;
            }
        }
        return ans;
    }
};
```

### 350. 两个数组的交集 II
第一个数组放入map，遍历第二个数组，在map中找当前元素出现过的次数；如果大于0，证明有交集，这时候输出到ans中，并且让出现的次数-1。

```C++
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int,int> map;
        vector<int> ans;

        for(auto n:nums1){
            map[n]++;
        }

        for(auto n:nums2){
            if(map[n] > 0){
                map[n]--;
                ans.push_back(n);
            }
        }
        return ans;
    }
};
```

### 128. 最长连续序列
https://leetcode-cn.com/problems/longest-consecutive-sequence/

用一个map记录当前位置元素所在序列的长度，然后每次遍历一个元素的时候，如果它的长度为0，就说明它是第一次出现，取其左右连续的元素的长度，将他们的长度相加再加上自身长度（+1）。

然后更新map中当前位置记录的长度、左右边界记录的最大长度，以及序列的最长长度。

```C++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }
        int ans = 0;

        unordered_map<int,int> map;
        int left=0,right=0;
        for(auto n:nums){
            int currentLength = 0;
            if(map[n] == 0){
                left = map[n-1];
                right = map[n+1];
                currentLength = 1+left+right;

                ans = max(ans,currentLength);
                map[n] = currentLength;
                map[n-left] = currentLength;
                map[n+right] = currentLength;
            }
        }
        return ans;
    }
};
```