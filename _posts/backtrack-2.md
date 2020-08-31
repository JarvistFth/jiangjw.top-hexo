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

//优化一下不用visited矩阵
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(board.size() == 0){
            return false;
        }

        // vector<vector<bool>> visited(board.size(),vector<bool>(board[0].size(),false));

        bool ans = false;

        int index = 0;

        for(int i=0;i<board.size();i++){
            for(int j=0;j<board[i].size();j++){
                ans = ans || dfs(board,word,index,i,j);
            }
        }
        return ans;


    }

    bool dfs(vector<vector<char>>& board, string& word, int index, int x, int y){

        if(index >= word.size()){
            return true;
        }

        if(x<0 || x>=board.size() || y<0 || y>=board[0].size() || board[x][y]=='0' || board[x][y]!=word[index]){
            return false;
        }

        char temp = board[x][y];
        board[x][y] = '0';

        if(dfs(board,word,index+1,x+1,y) || dfs(board,word,index+1,x-1,y) ||
            dfs(board,word,index+1,x,y+1) || dfs(board,word,index+1,x,y-1)){
                return true;
        }
        board[x][y] = temp;
        return false;


    }

    
};

```

# 980. 不同路径 III
迷宫类问题，这里难点在于需要经过每个‘0’的位置，所以需要维护一个变量统计步数；当我们走到dst的时候，判断一下步数是不是为0的个数。如果是，就认为路径+1；否则认为路径不合要求，不予统计。

这里本应该再用一个visited矩阵为是否走过提供标记作用；但是可以直接修改grid的值用做visited矩阵。

```C++
class Solution {
public:
    int uniquePathsIII(vector<vector<int>>& grid) {
        int step = 1;
        int x=0,y=0;

        for(int i=0;i<grid.size();i++){
            for(int j=0;j<grid[0].size();j++){
                if(grid[i][j] == 1){
                    x=i;y=j;
                }
                else if(grid[i][j] == 0){
                    step++;
                }
            }
        }
        return backtrack(grid,x,y,step);
    }

    int backtrack(vector<vector<int>>& grid, int x, int y, int step){
        if(x<0 || x>=grid.size() || y<0 || y>=grid[x].size() || grid[x][y] == -1){
            return 0;
        }

        if(grid[x][y] == 2){
            return step==0?1:0;
        }

        grid[x][y] = -1;
        int ans = backtrack(grid,x-1,y,step-1) + backtrack(grid,x+1,y,step-1) + backtrack(grid,x,y-1,step-1) + backtrack(grid,x,y+1,step-1);
        grid[x][y] = 0;
        return ans;

    }
};
```
