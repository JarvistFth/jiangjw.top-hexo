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

## 模板：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200914231021.png)

## 739. 每日温度
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

## 402. 移掉K位数字
https://leetcode-cn.com/problems/remove-k-digits/

我们需要将数字尽量按照从小到大保留，每次删除都尽量删掉位置比较高且数字比较大的数字。
比如“42351”，从左往右扫描，如果发现右边有比自己小的数字，就把它删除。

这就符合了单调栈的使用方法，要找到下一个比自己小的，维持一个单调增栈。

所以从左往右扫描，如果碰到比栈顶小的，如果还能删（k>0）就把栈顶的元素删掉（这是因为它的位置比较高，且数字也比较大，这样删掉的是最优解）。

但是如果已经是单调递增了，我们还要删除，就直接从栈顶开始删除就可以了（栈顶对应字符串末尾开始）。

最后从单调增栈构造出返回的字符串，由于是反过来的字符串，所以我们需要reverse。同时，在reverse之前，我们要去掉末尾的前导零。

```C++
class Solution {
public:
    string removeKdigits(string num, int k) {
        if(num.empty()){
            return "";
        }

        string ans = "";

        //less stack
        stack<char> stack;

        //find the next less num, delete the first num larger than the next num
        for(auto c:num){
            while(!stack.empty() && c < stack.top() && k>0){
                stack.pop();
                k--;
            }
            stack.push(c);
        }

        //stack here is less
        while(k){
            stack.pop();
            k--;
        }


        while(!stack.empty()){
            ans.push_back(stack.top());
            stack.pop();
        }
        //去掉前导0
        while(!ans.empty() && ans.back() == '0'){
            ans.pop_back();
        }

        if(ans.empty()){
            return "0";
        }

        reverse(ans.begin(),ans.end());
        return ans;

    }
};
```