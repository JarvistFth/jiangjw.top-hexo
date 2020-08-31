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


        vector<string> board(n,string(n,"."));
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

        //对当前字符串，从start到i取子串；后续递归从start+i开始取子串。
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

# 1286. 字母组合迭代器
https://leetcode-cn.com/problems/iterator-for-combination/

排列组合问题，第一次选择是从character长度中任意位置选一位；后面选完后，对后续的字符继续递归回溯选择。

当选定的字符串长度是给定长度时结束递归回溯。

```C++
class CombinationIterator {
public:
    CombinationIterator(string characters, int combinationLength) {
        string track = "";
        this->combinationLength = combinationLength;
        backtrack(characters,track,0);
    }
    
    string next() {
        return charactersArray[index++];
        
    }
    
    bool hasNext() {
        return index < charactersArray.size();
    }

    void backtrack(string& characters, string& track, int start ){
        if(track.size() == combinationLength){
            charactersArray.push_back(track);
            return;
        }

        for(int i=start;i<characters.size();i++){
            track.push_back(characters[i]);
            backtrack(characters,track,i+1);
            track.pop_back();
        }
    }


private:
    vector<string> charactersArray;
    int combinationLength;
    int index = 0;
};

/**
 * Your CombinationIterator object will be instantiated and called as such:
 * CombinationIterator* obj = new CombinationIterator(characters, combinationLength);
 * string param_1 = obj->next();
 * bool param_2 = obj->hasNext();
 */

 ```

 # 面试题 08.08. 有重复字符串的排列组合
 https://leetcode-cn.com/problems/permutation-ii-lcci/
 
 先排序，这样可以比较好的处理有重复的字符串。

 然后就是对每个字符做选择，选择完以后，递归回溯继续选择；

 当生成的字符串到达给定长度的时候，就结束递归；为防止重复选择，加入visited数组；难点在于通过式子：i>0 && s[i] == s[i-1] && !visited[i-1]，排除重复的字符串。

 利用这个式子，可以取两个重复字符串的左边。 意思是，当我们遍历到s[i]的时候，我们看看s[i]是不是和之前的一样；如果是的话，再看s[i-1]用过没有。如果s[i-1]用过了，就不使用；否则继续使用。但是我们从左侧开始遍历，所以这时候第一个重复的字母可以使用，而第二个使用的字母不可以使用。

 ```C++
 class Solution {
public:
    vector<string> ans;
    string path;
    int len = 0;
    vector<string> permutation(string S) {
        if(S.empty()){
            return {};
        }
        len = S.size();
        vector<bool> visited(len,false);
        sort(S.begin(),S.end());
        backtrack(S,path,visited);
        return ans;


    }

    void backtrack(string& s, string &path, vector<bool>& visited ){
        if(path.size() == len){
            ans.push_back(path);
            return ;
        }

        for(int i=0;i<len;i++){
            if(visited[i]){
                continue;
            }

            if(i>0 && s[i] == s[i-1] && !visited[i-1]){
                continue;
            }
           
            path.push_back(s[i]); 
            visited[i] = true;
            backtrack(s,path,visited);
            visited[i] = false;
            path.pop_back();
            
        }
    }
};
```

# 39. 组合总和
https://leetcode-cn.com/problems/combination-sum/


1. 当递归到sum==target时退出递归；
2. 对数组中每个数字做选择；
3. 从当前位置开始递归回溯继续选择数组； （所以要传入上次遍历的pos作为参数开始遍历）
4. 回溯时候要记得减去加上的sum。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        if(candidates.empty()){
            return {};
        }
        sort(candidates.begin(),candidates.end());
        backtrack(candidates,target,0,0);
        return ans;
    }

    void backtrack(vector<int>& candidates, int target, int sum, int start){
        if(sum == target){
            ans.push_back(path);
            return ;
        }

        for(int i=start;i<candidates.size();i++){
            if(candidates[i] + sum > target){
                break;
            }
            path.push_back(candidates[i]);
            sum+=candidates[i];
            backtrack(candidates,target, sum,i);
            path.pop_back();
            sum-=candidates[i];
        }
    }


};
```

# 1239. 串联字符串的最大长度

位运算奇巧淫技实在是学不会，，其实思想就是个bitmap，但是就是很难去使用。。。

所以构建个vector作为类哈希表使用；当vector对应字符下标存储的次数>1时，认为是有重复字符，就跳过递归选择。

照样的，for循环做第一次选择，递归做后面所有的选择；
每次做选择之前，都先判断一下有没有重复字符；如果有，跳过；否则继续。

如果没有重复字符， 每次都更新一下串联字符的最大长度，然后继续递归；递归前后回溯hash表的数值。

```C++
class Solution {
public:
    int ans = 0;
    int maxLength(vector<string>& arr) {
        vector<int> hash(26);
        backtrack(arr,hash,0,0);
        return ans;
    }

    void backtrack(vector<string>& a, vector<int>& hash, int currentLen, int start){
        

        for(int &i:hash){
            if(i>1){
                return;
            }
        }
        
        ans = max(ans,currentLen);

        
        for(int i=start;i<a.size();i++){
            
            for(auto& s:a[i]){
                hash[s - 'a']++;
            }
            backtrack(a,hash,currentLen+a[i].size(),i+1);
            for(auto& s:a[i]){
                hash[s - 'a']--;
            }
        }
    }
};

```