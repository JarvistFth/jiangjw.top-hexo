---
title: leetcode-slidingwindow
date: 2020-08-19 21:18:11
categories: leetcode-slidingwindow
tags: [leetcode ,滑动窗口]
keywords: [leetcode,滑动窗口]
---

参考labudalong大佬的滑动窗口框架；一些是适用于字符串，一些则不是。

<!---more--->

滑动窗口框架：
```C++
//普通版：
int left=0,right=0;
        while(right<A.size()){
            //移入窗口的元素
            //第一步：入窗：
            int r = A[right];
            //判断条件，对窗口内的数据进行处理更新
            if(r == 0){
                // todo
            }
            //第二步：出窗：
            //判断窗口需要左移的条件
            while(window needs shrink){
                //移出窗口的元素
                int l = A[left];
                //对窗口内数据进行更新
                if(l == 0) {
                    // todo
                }
                //窗口左移
                left++;
            }
            //第三步：干该干的
            //一般是看情况在哪里更新答案的长度，看结果应该在扩大窗口还是缩小窗口时候进行更新
            // ans = max(ans,right-left+1);
            //窗口右移
            right++;
        }

        return ans;



class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char,int> need,window;

        int left=0,right=0;
        //
        int valid = 0;

        for(auto c:t){
            need[c]++;
        }

        while(right<s.size()){
            //元素移入窗口
            char c = s[right];

            //对窗口内元素进行处理
            if(need.count(c)){
                window[c]++;
                if(window[c] == need[c]){
                    valid++;
                }
            }

            //判断元素移出窗口
            while(valid == need.size()){

                //d是要移出的元素
                char d = s[left];
                //窗口左移
                left++;
                //对窗口内元素进行处理
                if(need.count(d)){
                    if(window[d] == need[d]){
                        valid--;
                    }
                    window[d]--;
                }
            }
            right++;
        }
        // return len==INT_MAX?"":s.substr(start,len);
    }
};        


```

# 剑指 Offer 48. 最长不含重复字符的子字符串
https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/

用一个map维护一个window，窗口右移。当window里面是有遍历的字符串的时候，窗口左移。左移后比较当前长度是否是最长。

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char,int>window;

        int left=0,right=0;
        int ans = 0;

        while(right<s.size()){
            char c = s[right];
            window[c]++;
            while(window[c] > 1){
                char d = s[left];
                left++;
                window[d]--;
            }
            ans = max(ans,right-left+1);
            right++;
        }
        return ans;
    }
};
```

# 1004. 最大连续1的个数 III
https://leetcode-cn.com/problems/max-consecutive-ones-iii/
窗口右移；当遇到0的个数大于K时，窗口左移。在窗口左移后更新最大长度。

```C++
class Solution {
public:
    int longestOnes(vector<int>& A, int K) {
        int ans = 0;

        int left=0,right=0;
        int zeroCount = 0;
        while(right<A.size()){
            int r = A[right];
            
            if(r == 0){
                zeroCount++;
            }
            while(zeroCount > K){
                if(A[left] == 0) {
                    zeroCount--;
                }
                left++;
            }
            
            ans = max(ans,right-left+1);
            right++;
        }

        return ans;
    }
};
```

# 480. 滑动窗口中位数
https://leetcode-cn.com/problems/sliding-window-median/

想法就是用一个vector维护一个window，每次window超过K个元素就出窗；入窗的时候通过二分查找维护窗口有序插入，这样可以方便找到window的中位数(window[(i-1)/2] + window(i/2))/2 。
代码比较简单，也比较容易理解，但是复杂度比较高O(n^2)级别了应该【窗口遍历O(n)，插入排序O(n)】。

```C++
class Solution {
public:
    vector<double> medianSlidingWindow(vector<int>& nums, int k) {

        int left=0,right=0;
        vector<double>window;
        vector<double>ans;

        while(right<nums.size()){

            auto index = lower_bound(window.begin(),window.end(),nums[right]);
            window.insert(index,nums[right]);

            while(window.size()>=k){
                if(window.size() == k){
                    double mid = (window[(k-1)/2] + window[k/2])/2.0;
                    ans.push_back(mid);
                }

                auto index = lower_bound(window.begin(),window.end(),nums[left]);
                window.erase(index);

                left++;
            }
            right++;  
        }

        return ans;


    }
};
```

# 76. 最小覆盖子串
https://leetcode-cn.com/problems/minimum-window-substring/

字符串子串的滑动窗口模板。维护一个window和一个need，入窗时判断一下入窗的元素是不是子串的字符，如果是window++;如果window和need维护的字符次数相同，就说明window这个字符可用；当window维护的可用字符==need.size()，说明子串和原字符串已经完全覆盖了。这时候为了保证长度最小，就要窗口左移，元素出窗。出窗的时候也要更新窗口内的数据。同时更新最小长度。

```C++
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char,int> need,window;

        int left=0,right=0;
        int start=0,len=INT_MAX;
        int valid = 0;

        for(auto c:t){
            need[c]++;
        }

        while(right<s.size()){
            char c = s[right];
            if(need.count(c)){
                window[c]++;
                if(window[c] == need[c]){
                    valid++;
                }
                

            }

            while(valid == need.size()){
                if(right-left+1<len){
                    start = left;
                    len = right-left+1;
                }

                char d = s[left];
                left++;
                if(need.count(d)){
                    if(window[d] == need[d]){
                        valid--;
                    }
                    window[d]--;
                }
            }
            right++;

        }

        return len==INT_MAX?"":s.substr(start,len);
    }
};
```

# 239. 滑动窗口最大值
https://leetcode-cn.com/problems/sliding-window-maximum/

```C++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {


        vector<int> ans;
        deque<int> window;
        int right=0;
        

        while(right<nums.size()){
            if(!window.empty() && window.front() == right-k){
                window.pop_front();
            }
            while(!window.empty() && nums[right]>nums[window.back()]){
                window.pop_back();
            }
            window.push_back(right);
            if(right>=k-1){
                ans.push_back(nums[window.front()]);
            }
            right++;

        }
        return ans;
    }
};
```