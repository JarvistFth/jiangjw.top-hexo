---
title: leetcode-贪心算法系列（2）
date: 2020-09-07 22:31:21
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---

贪心问题，按照通过率排序，第二页题目。

<!---more--->

## 1518. 换酒问题
https://leetcode-cn.com/problems/water-bottles/

有多少空的就换多少瓶；换到空的瓶子小于可以兑换数量为止。

```C++
class Solution {
public:
    int numWaterBottles(int numBottles, int numExchange) {
        int ans = numBottles;
        int emptyBottles = numBottles;

        while(emptyBottles >= numExchange){
            int get = emptyBottles/numExchange;
            ans += get;
            emptyBottles = get + (emptyBottles - (get * numExchange));
        }
        return ans;

    }
};
```

## 406. 根据身高重建队列
https://leetcode-cn.com/problems/queue-reconstruction-by-height/

先将数组按照身高倒序排序，这样的好处是往排好队的位置插入的时候，如果要插队的人比前面的人矮，就可以直接插入，不用处理其他元素。其他人插队的时候，直接按照其所在的位置k（p[1]）插入就可以了。

```c++
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

## 1029. 两地调度
https://leetcode-cn.com/problems/two-city-scheduling/

先假设所有人去A，然后将所有人去A和去B的花费差按照倒序排序，这样表示前半部分的人应该去B；

所以让前半部分的人去B，原来的花费和减去花费差就是答案。

```C++
class Solution {
public:
    int twoCitySchedCost(vector<vector<int>>& costs) {
        int ans = 0;

        for(auto &c:costs){
            ans += c[0];
        }

        vector<int> bfee;

        for(auto &c:costs){
            bfee.push_back(c[0] - c[1]);
        }
        sort(bfee.begin(),bfee.end(),[](int a, int b){
            return a>b;
        });

        for(int i=0;i<bfee.size()/2;i++){
            ans -= bfee[i];
        }
        return ans;
    }
};
```

## 1338. 数组大小减半
https://leetcode-cn.com/problems/reduce-array-size-to-the-half/

用map统计每个元素出现的次数，然后将次数排序，每次都从出现次数最多的删起；当删掉的长度比原长度一半要大，就return。

```C++
class Solution {
public:
    int minSetSize(vector<int>& arr) {
        unordered_map<int,int> map;

        vector<int> count;

        for(auto i:arr){
            map[i]++;
        }

        for(auto &[k,v] : map){
            count.push_back(v);
        }

        sort(count.begin(),count.end(), greater<int>());

        int len = 0, size = arr.size();

        for(int i=0;i<count.size();i++){
            len += count[i];
            if(len >= size/2){
                return i+1;
            }
        }
        return -1;
    }
};
```

## 1433. 检查一个字符串是否可以打破另一个字符串
https://leetcode-cn.com/problems/check-if-a-string-can-break-another-string/

先排序，然后按顺序比较，看看每个位置是不是有大有小；如果是，就说明不能打破字符串，返回false。

```C++
class Solution {
public:
    bool checkIfCanBreak(string s1, string s2) {
        sort(s1.begin(),s1.end());
        sort(s2.begin(),s2.end());
        bool greater = false, less = false;

        for(int i=0;i<s1.size();i++){
            if(s1[i] == s2[i]){
                continue;
            }
            if(s1[i] > s2[i]){
                greater = true;
            }else{
                less = true;
            }
            if(greater && less){
                return false;
            }
        }
        return true;
    }
};
```