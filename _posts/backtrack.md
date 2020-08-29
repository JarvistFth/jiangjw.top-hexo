---
title: 回溯算法-排列组合问题
date: 2020-08-28 23:57:19
categories: 
- leetcode
- 回溯
tags: [leetcode ,回溯]
keywords: [leetcode,回溯]
---

回溯算法！！

由labuladong大佬的算法框架，整体框架如下：
```Python

def backtrack(path, choice):
    if endBacktrack:
        result.add(path)
        return
    
    for choose in choice:
        make a choice
        backtrack(path,choice)
        undo the choice

```


<!---more--->

# N皇后问题
https://leetcode-cn.com/problems/eight-queens-lcci/

递归枚举每一行，对每个位置都做选择（放置皇后Q），路径就是整个棋盘。

```C++
class Solution {
public:
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int n) {


        vector<string> board(n,string(n,'.'));
        backtrack(board,0);
        return ans;

    }

    void backtrack(vector<string>& board, int row){
        if(row == board.size()){
            ans.push_back(board);
            return ;
        }

        for(int col=0;col<board[row].size();col++){
            if(!canSet(board,row,col)){
                continue;
            }

            board[row][col] = 'Q';
            backtrack(board,row+1);
            board[row][col] = '.';
        }
    }

    bool canSet(vector<string>& board,int row,int col){
        for(int i=0;i<board.size();i++){
            if(board[i][col] == 'Q'){
                return false;
            }
        }
        
        //zuoshangjiao
        for(int i=row-1,j=col-1; i>=0 && j>=0 ;i--,j--){
            if(board[i][j] == 'Q'){
                return false;
            }
        }

        //youshangjiao
        for(int i=row-1, j=col+1; i>=0 && j<board.size();i--,j++){
            if(board[i][j] == 'Q'){
                return false;
            }
        }
        return true;

    }
};
```

# 131. 分割回文串
https://leetcode-cn.com/problems/palindrome-partitioning/

对字符串s，从起点到末尾，每一位都要获取其子串；

对每一位获取了子串的后续字符串，也要递归地继续从前一子串的后面开始继续获取子串。

每次获取了子串，就看一下他符不符合条件，如果符合，1.就加入到path中；2.然后递归调用继续获取后续子串，3.pop。


```C++
class Solution {
public:
    vector<vector<string>> partition(string s) {
        vector<vector<string>> ans;
        vector<string> subset;

        backtrack(s,0,s.size(),ans,subset);
        return ans;
    }

    bool isValid(const string& s){
        int i=0,j=s.size()-1;

        while(i<j){
            if(s[i] != s[j]){
                return false;
            }
            i++;j--;
        }
        return true;
    }

    void backtrack(const string& s, int start, int len, vector<vector<string>>& ans, vector<string>& subset){
        if(start>=len){
            ans.push_back(subset);
            return;
        }

        for(int i=1;i<=len-start;i++){
            string s1 = s.substr(start,i);
            if(!isValid(s1)){
                continue;
            }
            subset.emplace_back(std::move(s1));
            backtrack(s,start+i,len,ans,subset);
            subset.pop_back();
        }
    }
};
```

# 1415. 长度为 n 的开心字符串中字典序第 k 小的字符串
https://leetcode-cn.com/problems/the-k-th-lexicographical-string-of-all-happy-strings-of-length-n/

枚举3个字符，对三个字符做选择；当前一个字符和当前字符相等的时候，不做递归回溯。

递归到n为0为止，将当前的string放入ans。

```C++
class Solution {
public:
    string getHappyString(int n, int k) {
        vector<string> ans;
        string s;
        char last = 0;
        backtrack(n,k,last,s,ans);
        return ans.size()<k?"":ans[k-1];
    }

    void backtrack(int n, int k, char last, string& s, vector<string>& ans){
        if(n == 0){
            ans.push_back(s);
            return;
        }

        for(char c = 'a'; c<='c';c++){
            if(last == c){
                continue;
            }
            s.push_back(c);
            backtrack(n-1,k,c,s,ans);
            s.pop_back();
        }
    }
};
```