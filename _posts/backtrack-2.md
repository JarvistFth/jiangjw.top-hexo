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

# 37. 解数独
https://leetcode-cn.com/problems/sudoku-solver/

和N皇后问题有一点点相似：

但是N皇后问题是排列组合，这里只有唯一解。所以每次去递归回溯的时候，要判断一下当前选择可不可行，如果是条死路，就要进行回溯了。

难点在于确定每个小的九宫格里面的数字唯一性判断，题解是用了board[row/3*3 + i/3][col/3*3 + i%3]来获得每个小九宫格的横竖坐标起始值，然后用变量i去控制遍历整个小的九宫格。

整个流程可以分为：

1. 对当前是空格的位置，从1-9选取字符填入，不是空格字符，跳过，从当前行下一列开始填数；
2. 判断一下当前这个字符可不可以填入（行、列、小九宫格是否有重复的）；如果有重复，不可填入；
3. 尝试填入该数；
4. 尝试递归机修填入下一个数，这里以下一列为单位；
5. 如果下一列填不进数，会发现这是条错误的选择，返回false；
6. 回溯，撤销当前填入的数；
7. 当列遍历到第九列时，开始往下一行填数，递归下一行；
8. 当遍历到最后一行的时候（这时前八行也已经填好了），结束递归返回true。


```C++
class Solution {
public:
    void solveSudoku(vector<vector<char>>& board) {
        if(board.empty() || board.size() != 9){
            return ;
        }
        backtrack(board,0,0);


    }

    bool backtrack(vector<vector<char>>& board, int row, int col){
        if(col == 9){
            return backtrack(board,row+1,0);
        }

        if(row == 9){
            return true;
        }

        if(board[row][col] != '.'){
            return backtrack(board,row,col+1);
        }

        for(char c = '1'; c <= '9'; c++){
            if(!isValid(board,row,col,c)){
                continue;
            }
            board[row][col] = c;
            if(backtrack(board,row,col+1)){
                return true;
            }
            board[row][col] = '.';
        }
        return false;

    }

    bool isValid(vector<vector<char>>& board, int row, int col, char c){
        for(int i=0;i<9;i++){
            if(board[i][col] == c){
                return false;
            }
            if(board[row][i] == c){
                return false;
            }
            if(board[row/3*3 + i/3][col/3*3 + i%3] == c){
                return false;
            }
        }
        return true;
    }
};
```

# 1219. 黄金矿工

迷宫类题目，这里的起点是矩阵内任意起点，所以对每一个点都要做选择和递归回溯。

1. 迷宫类问题递归回溯第一步，确定边界返回条件：当前坐标超出矩阵范围，或者不可选择或者已经访问过（grid[x][y] == 0），返回0；
2. 开始选择，先保存当前grid[x][y]的值，然后进行递归选择；
3. 撤销选择，结束回溯；
4. 递归选择返回的是下一步四个方向的最大值，因此总最大值应该是当前节点的值和返回的最大值，返回总最大值。


```C++
class Solution {
public:
    int ans = 0;
    int getMaximumGold(vector<vector<int>>& grid) {
        if(grid.empty()){
            return 0;
        }
        for(int i=0;i<grid.size();i++){
            for(int j=0;j<grid[i].size();j++){
                if(grid[i][j] <= 0){
                    continue;
                }
                ans = max(ans,backtrack(grid,i,j));
            }
        }
        return ans;
    }

    int backtrack(vector<vector<int>>& grid, int x, int y){
        if(x<0 || x>=grid.size() || y<0 || y>=grid[x].size() || grid[x][y] == 0){
            return 0;
        }

        int temp = grid[x][y];
        grid[x][y] = 0;
        int maxval = max({backtrack(grid,x+1,y), backtrack(grid,x-1,y), backtrack(grid,x,y-1), backtrack(grid,x,y+1)});
        grid[x][y] = temp;
        return temp+maxval;
    }
};
```