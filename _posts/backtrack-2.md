---
title: 回溯算法-迷宫类问题
date: 2020-08-29 21:47:50
categories: 
- leetcode
- 回溯
tags: [leetcode ,回溯]
keywords: [leetcode,回溯]
---

回溯算法迷宫类问题，一般涉及一个visited二维数组，可以从某个起点开始，走向多个方向；遇到不符合条件的，回到上一步；否则继续往下一步走。
<!---more--->

这里的单词搜索就像是走迷宫类的题目，从任意起点开始，往上下左右四个方向匹配字符串。

匹配过程中，如果越界，返回false；然后标记visited数组，表示当前已经走过，防止走回头路；然后就是递归dfs了；结束递归后取消选择（标记）。

# 79. 单词搜索
https://leetcode-cn.com/problems/word-search/


```C++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(board.size() == 0){
            return false;
        }

        vector<vector<bool>> visited(board.size(),vector<bool>(board[0].size(),false));

        bool ans = false;

        int index = 0;

        for(int i=0;i<board.size();i++){
            for(int j=0;j<board[i].size();j++){
                ans = ans || dfs(board,word,visited,index,i,j);
            }
        }
        return ans;


    }

    bool dfs(vector<vector<char>>& board, string& word, vector<vector<bool>>& visited, int index, int x, int y){

        if(index >= word.size()){
            return true;
        }

        if(x<0 || x>=board.size() || y<0 || y>=board[0].size() || visited[x][y] || board[x][y]!=word[index]){
            return false;
        }

        visited[x][y] = true;

        if(dfs(board,word,visited,index+1,x+1,y) || dfs(board,word,visited,index+1,x-1,y) ||
            dfs(board,word,visited,index+1,x,y+1) || dfs(board,word,visited,index+1,x,y-1)){
                return true;
        }
        visited[x][y] = false;
        return false;


    }

    
};
```
