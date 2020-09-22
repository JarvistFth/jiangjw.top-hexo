---
title: monotoneStack
date: 2020-09-14 23:00:13
categories:
- leetcode
- 单调栈
tags: [leetcode ,单调栈]
keywords: [leetcode,单调栈]
---
单调栈实际上就是栈，只是利⽤了⼀些巧妙的逻辑，使得每次新元素⼊栈
后，栈内的元素都保持有序（单调递增或单调递减）。

单调栈⽤途不太⼴泛，只处理⼀种典型
的问题，叫做 Next Greater Element。

————节自labuladong的算法笔记。
<!---more--->

# 模板：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200914231021.png)

# 739. 每日温度
https://leetcode-cn.com/problems/daily-temperatures/

这里放入栈的应该是下一个更大数字的索引。

```C++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& T) {
        if(T.empty()){
            return {};
        }
        
        stack<int> stack;
        vector<int> ans(T.size());

        for(int i=T.size()-1;i>=0;i--){
            while(!stack.empty() && T[stack.top()] <= T[i]){
                stack.pop();
            }
            ans[i] = stack.empty()?0:stack.top() - i;
            stack.push(i);
        }
        return ans;
    }
};
```
