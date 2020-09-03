---
title: 回溯-杂题
date: 2020-09-03 21:40:03
categories: 
- leetcode
- 回溯
tags: [leetcode ,回溯]
keywords: [leetcode,回溯]
---


# 22. 括号生成
https://leetcode-cn.com/problems/generate-parentheses/

用两个变量记录剩下可以用的括号数量，然后我们不断做选择，加入左括号或者右括号，递归回溯，当可以用的括号数目小于0时，退出；

如果左边可以用的数目是大于右边的，这时候说明我们先选了较多的右括号，这时候即使选到最后，也会有左括号和右括号相反，所以是不可行的，也return。

如果刚好左右都用完，就证明完全匹配，加入ans。

```C++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> ans;
        string path;
        backtrack(ans,path,n,n,n);
        return ans;
    }

    void backtrack(vector<string>& ans, string& path, int n, int left, int right){
        if(left < 0 || right < 0){
            return ;
        }
        if(left > right){
            return ;
        }
        if(left == 0 && right == 0){
            ans.push_back(path);
            return ;
        }

        path.push_back('(');
        backtrack(ans,path,n,left-1,right);
        path.pop_back();

        path.push_back(')');
        backtrack(ans,path,n,left,right-1);
        path.pop_back();

    }
};
```