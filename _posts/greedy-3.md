---
title: greedy-3
date: 2020-09-09 21:11:34
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---

贪心问题，按照通过率排序，第三页题目。

<!---more--->

## 1046. 最后一块石头的重量
https://leetcode-cn.com/problems/last-stone-weight/

直觉是排序，但是排序后还要对操作后的数值继续保持有序，所以感觉直接用vector不太好用。

直觉又告诉我，需要一个push进去元素以后也保持有序的数据结构，要么是map要么是priorityqueue。

这里是取最大的两个，所以显然是用priorityqueue。建大根堆，取最大的两个，碰撞，碰撞后体积继续推入堆。

直到堆中元素小于2；如果堆为空，返回0；否则返回剩下元素的重量。

```C++
class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        if(stones.empty()){
            return 0;
        }

        priority_queue<int,vector<int>,less<int>> queue;
        for(auto s:stones){
            queue.push(s);
        }

        while(queue.size() >= 2){
            int s1 = queue.top();
            queue.pop();
            int s2 = queue.top();
            queue.pop();
            int left = s1 - s2;
            if(left > 0){
                queue.push(left);
            }
        }
        if(queue.empty()){
            return 0;
        }
        return queue.top();

    }
};
```

## 1558. 得到目标数组的最少函数调用次数
https://leetcode-cn.com/problems/minimum-numbers-of-function-calls-to-make-target-array/

反着推，从现有的数字，分别进行-1和/2操作，看多少步可以到0.

首先是对所有的奇数都做-1，然后对-1后的所有的偶数做/2，这样可以最快缩小到1，这时候再-1就可以了。

所以我们统计一下0的个数是不是数组的长度，如果是，就退出；
如果不是，我们继续进行变换。

每次变换都是优先判断是不是奇数，如果是奇数，就将他做-1的变换，这时操作次数+1；操作完以后，每个都是偶数，对偶数做/2；这样操作完一遍后，我们统计一下0的个数，如果0的个数不是数组长度，证明这时候/2的变换做了，但是还没能让全部元素变成0，所以这时候操作数还要+1 。

```C++
class Solution {
public:
    int minOperations(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }

        int ans = 0;

        int zeroNum = 0;

        while(zeroNum != nums.size()){
            zeroNum = 0;
            for(auto& n:nums){
                if(n % 2 == 1){
                    ans++;
                    n--;
                }
                n=n/2;
                if(n == 0){
                    zeroNum++;
                }
            }
            if(zeroNum < nums.size())
                ans++;
        }
        return ans ;
    }
};
```

## 1578. 避免重复字母的最小删除成本
https://leetcode-cn.com/problems/minimum-deletion-cost-to-avoid-repeating-letters/

发现两个字母相等的时候，取成本低的那个，如果是靠后的成本较低的话，我们取完他以后，让他更新为前一个成本的值，用于后面进行比较，留下成本最高的不取。

```C++
class Solution {
public:
    int minCost(string s, vector<int>& costs) {
        if(s.empty() || costs.empty()){
            return 0;
        }

        int ans = 0;
        for(int i=1;i<costs.size();i++){
            if(s[i] == s[i-1]){
                if(costs[i-1] > costs[i]){
                    ans+=costs[i];
                    costs[i] = costs[i-1];
                }else{
                    ans += costs[i-1];
                }
            }
        }
        return ans;
    }
};
```

## 1094. 拼车
https://leetcode-cn.com/problems/car-pooling/

用一个vector表示每个车站上下车的人数，对vector求和就表示这个车上的总人数。只要某个区间内乘客的人数大于capacity，就return false。

上车就是该起点人数增加，下车就是该终点人数减少。对起点和终点求和就是车上人数。

```C++
class Solution {
public:
    bool carPooling(vector<vector<int>>& trips, int capacity) {
        if(trips.empty()){
            return true;
        }

        vector<int> passenger(1010);

        for(auto &t:trips){
            passenger[t[1]] += t[0];
            passenger[t[2]] -= t[0];
        }
        int total = 0;
        for(auto p:passenger){
            total += p;
            if(total > capacity){
                return false;
            }
        }
        return true;
    }
};
```

## 452. 用最少数量的箭引爆气球
https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/

先按右端点排序，然后判断下一个的左端点是不是在右端点的右侧，如果是，证明还需要多射一支箭；

如果不是，因为这时候已经按右端点排序了，所以射箭的位置肯定经过下一个气球，就不需要多射一支箭。

```C++
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        if(points.empty()){
            return 0;
        }

        sort(points.begin(),points.end(),[](vector<int>& a, vector<int>& b){
            return a[1]<b[1];
        });

        int x = points[0][1];
        int ans = 1;

        for(int i=1;i<points.size();i++){
            if(points[i][0] > x){
                ans++;
                x = points[i][1];
            }
        }
        return ans;
    }
};
```

## 1400. 构造 K 个回文字符串
https://leetcode-cn.com/problems/construct-k-palindrome-strings/

要构造最多的回文字符串，就让一个奇数字符作为一个回文。 每个奇数，必在k中占一个名额。 所以odds <= k; 其他的是偶数元素，构造的话随便安排。

```C++
class Solution {
public:
    bool canConstruct(string s, int k) {
        if(s.size() < k){
            return false;
        }
        vector<int> dict(26);

        for(auto c:s){
            dict[c - 'a']++;
        }
        int odd = 0;
        for(auto d:dict){
            if(d % 2 == 1){
                odd++;
            }
        }
        return odd <=k;
    }
};
```

## 1247. 交换字符使得字符串相同
https://leetcode-cn.com/problems/minimum-swaps-to-make-strings-equal/

s1和s2可以交换任意位置的字符，所以我们统计不相等的x和y的字符总数；

如果不相等的x或y的字符数为偶数，则每两个字符只要交换一次就可以让他们相等了。

如果不相等的x和y的字符数为1，则要交换两次。

如果要交换的x和y的和是奇数，就说明就算交换，也不能使得字符串相同，return -1。



```C++
class Solution {
public:
    int minimumSwap(string s1, string s2) {
        int countx = 0, county = 0;
        for(int i=0;i<s1.size();i++){
            if(s1[i] != s2[i]){
                if(s1[i] == 'x'){
                    countx ++;
                }else{
                    county ++;
                }
            }
        }
        if((countx + county) % 2 == 1){
            return -1;
        }
        return countx/2 + county/2 + 2*(countx % 2);
    }
};
```