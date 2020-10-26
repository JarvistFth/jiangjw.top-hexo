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

## N皇后问题
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




## 131. 分割回文串
https://leetcode-cn.com/problems/palindrome-partitioning/

对字符串s，从起点到末尾，每一位都要获取其子串；

对每一位获取了子串的后续字符串，也要递归地继续从前一子串的后面开始继续获取子串。

每次获取了子串，就看一下他符不符合条件，如果符合，1.就加入到path中；2.然后递归调用继续获取后续子串，3.pop。


```C++
class Solution {
public:
   
    vector<vector<string>> partition(string s) {
        vector<vector<string>> ans;
        vector<string> track;
        backtrack(s,0,ans,track);
        return ans;
    }
    
    //做选择，对s取子串，看是不是回文串
    void backtrack(string &s, int start, vector<vector<string>>& ans, vector<string>& subset){
         if(start>=s.size()){
            ans.push_back(subset);
            return;
        }
        string s1;
        for(int i=start;i<s.size();i++){
            s1 += s[i];
            if(!isValid(s1)){
                continue;
            }
            subset.push_back(s1);
            backtrack(s,i+1,ans,subset);
            subset.pop_back();
        }
    }

    bool isValid(string &s){
        int i=0, j=s.size()-1;
        while(i<j){
            if(s[i] != s[j]){
                return false;
            }
            i++;j--;
        }
        return true;
    }
};
```

## 1415. 长度为 n 的开心字符串中字典序第 k 小的字符串
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

## 1286. 字母组合迭代器
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

 ## 面试题 08.08. 有重复字符串的排列组合
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

## 39. 组合总和
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

## 1239. 串联字符串的最大长度

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


## 216. 组合总和 III
https://leetcode-cn.com/problems/combination-sum-iii/

枚举1-9的数字，从当前选择的数字后面开始继续递归选择。

当组合长度为k时退出递归；如果此时sum==n，就将当前组合发入ans。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combinationSum3(int k, int n) {
        backtrack(1,k,n,0);
        return ans;
    }

    void backtrack(int start,int k, int n, int sum){
        if(k == path.size()){
            if(sum == n){
                ans.push_back(path);
            }
            return ;
        }

        for(int i=start;i<=9;i++){
            path.push_back(i);
            sum+=i;
            backtrack(i+1,k,n,sum);
            path.pop_back();
            sum-=i;
        }
    }
};
```

## 面试题 08.04. 幂集
子集问题，和组合不一样的是，每次遍历一个数字，都可以放入答案中。

循环枚举和组合排列都是一样的。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<int> path;
        backtrack(path,nums,0);
        return ans;
    }

    void backtrack(vector<int>& path, vector<int>&nums, int start){
        ans.push_back(path);

        for(int i=start;i<nums.size();i++){
            path.push_back(nums[i]);
            backtrack(path,nums,i+1);
            path.pop_back();
        }
    }
};
```

## 面试题 08.07. 无重复字符串的排列组合
https://leetcode-cn.com/problems/permutation-i-lcci/

无重复字符串，所以不用通过s[i] == s[i-1] && !visited[i-1]来去重；

枚举每个字符串，为了防止重复枚举，需要一个visited数组来控制。

当递归回溯到当前字符串的长度和给定字符串长度时，结束递归。

```C++
class Solution {
public:
    vector<string> permutation(string S) {
        vector<string> ans;
        vector<bool> visited(S.size(),false);
        string path;
        backtrack(ans,path,S,visited);
        return ans;
    }

    void backtrack(vector<string>& ans, string& path, string& s, vector<bool>& visited ){
        if(path.size() == s.size()){
            ans.push_back(path);
            return ;
        }

        for(int i=0;i<s.size();i++){
            if(visited[i]){
                continue;
            }

            visited[i] = true;
            path.push_back(s[i]);
            backtrack(ans,path,s,visited);
            visited[i] = false;
            path.pop_back();
        }
    }
};
```

## 17. 电话号码的字母组合
https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

看到这一串映射先想到用map保存每一个char的对应关系；

原来是想用for去枚举每一个digits的位置，但是用for后再backtrack回溯，这时候还要继续用for来枚举后面的每一位数字对应的字符。

后来一想，这里的做选择其实是一步一步做的，选完了第一位数字，再选第二位数字，在选择数字的时候做回溯会比较好。

所以针对每一位数字做递归回溯选择，当选择的位置>=digits的长度的时候，退出递归。

进行数字的选择后，从哈希表中拿出数字对应的字符串，对字符串进行枚举，每一位字符都先选一个，然后进入到递归回溯环节。

```C++
class Solution {
public:
    unordered_map<char,string> map;
    vector<string> ans;
    vector<string> letterCombinations(string digits) {
        if(digits.empty()){
            return {};
        }
        string path;
        init();
        backtrack(digits,path,0);
        return ans;


    }

    void backtrack(string& digits, string& path, int start){
        if(start >= digits.size()){
            ans.push_back(path);
            return ;
        }

        char c = digits[start];
        string str = map[c];
        for(auto &s:str){
            path.push_back(s);
            backtrack(digits,path,start+1);
            path.pop_back();
        }
    }

    void init(){
        map['2'] = "abc";
        map['3'] = "def";
        map['4'] = "ghi";
        map['5'] = "jkl";
        map['6'] = "mno";
        map['7'] = "pqrs";
        map['8'] = "tuv";
        map['9'] = "wxyz";
    }
};
```

## 90. 子集 II
https://leetcode-cn.com/problems/subsets-ii/

子集每个path都要加入到ans中；

照样使用nums[i] == nums[i-1] && !visited[i-1]去重，一定要用!visited，虽然其他题用visited没问题，但是这道题会出问题。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        if(nums.empty()){
            return {};
        }
        sort(nums.begin(),nums.end());
        vector<int> path;
        vector<bool> visited(nums.size(),false);
        backtrack(nums,path,visited,0);
        return ans;
    }

    void backtrack(vector<int>& nums, vector<int>& path, vector<bool>& visited, int start){
        ans.push_back(path);
        if(start >= nums.size()){
            return ;
        }

        for(int i=start;i<nums.size();i++){
            if(visited[i]){
                continue;
            }
            if(i>0 && nums[i] == nums[i-1] && !visited[i-1]){
                continue;
            }

            path.push_back(nums[i]);
            visited[i] = true;
            backtrack(nums,path,visited,i+1);
            visited[i] = false;
            path.pop_back();
        }
    }
};
```

## 46. 全排列
https://leetcode-cn.com/problems/permutations/

老套路了，全排列只需要判断一下自身是否已经加入到集合中，需要个visited数组。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;

    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> visisted(nums.size(),false);
        backtrack(nums,visisted);
        return ans;
    }

    void backtrack(vector<int>& nums, vector<bool>& visisted){
        if(path.size() == nums.size()){
            ans.push_back(path);
            return;
        }

        for(int i=0;i<nums.size();i++){
            if(visisted[i]){
                continue;
            }

            visisted[i] = true;
            path.push_back(nums[i]);
            backtrack(nums,visisted);
            visisted[i] = false;
            path.pop_back();
        }
    }
};
```

## 784. 字母大小写全排列
https://leetcode-cn.com/problems/letter-case-permutation/

isalpha tolower toupper 操作字符的好东西啊。。。

基本想法和排列一样，选择一个字符，看他是不是字母，如果不是，直接递归回溯对下一个字符做选择；否则就分别选择他的小写和他的大写。

```C++
class Solution {
public:
    vector<string> ans;
    vector<string> letterCasePermutation(string S) {
        string path;
        backtrack(S,path,0);
        return ans;
    }

    void backtrack(string& s, string& path, int start){
        if(start == s.size()){
            ans.push_back(path);
            return ;
        }

        
        if('0' <= s[start] && s[start] <= '9'){
            path.push_back(s[start]);
            backtrack(s,path,start+1);
            path.pop_back();
        }else{
            path.push_back(tolower(s[start]));
            backtrack(s,path,start+1);
            path.pop_back();

            path.push_back(toupper(s[start]));
            backtrack(s,path,start+1);
            path.pop_back();
        }
    }
};
```

## 47. 全排列 II
https://leetcode-cn.com/problems/permutations-ii/

去重的排列，排序，i>0 && nums[i] == nums[i-1] && !visited[i-1]去重；

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        if(nums.empty()){
            return {};
        }
        sort(nums.begin(), nums.end());
        vector<bool> visited(nums.size(),false);
        backtrack(nums,visited);
        return ans;
    }

    void backtrack(vector<int>& nums, vector<bool>& visited){
        if(path.size() == nums.size()){
            ans.push_back(path);
            return ;
        }

        for(int i=0;i<nums.size();i++){
            if(visited[i]){
                continue;
            }

            if(i>0 && nums[i] == nums[i-1] && !visited[i-1]){
                continue;
            }
            visited[i] = true;
            path.push_back(nums[i]);
            backtrack(nums,visited);
            path.pop_back();
            visited[i] = false;
        }
    }

};
```

## 40. 组合总和 II
https://leetcode-cn.com/problems/combination-sum-ii/



```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int>path;
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        if(candidates.empty()){
            return {};
        }
        vector<bool> visited(candidates.size(),false);
        sort(candidates.begin(),candidates.end());
        backtrack(candidates,0,target,visited);
        return ans;
    }

    void backtrack(vector<int>& candidates, int start, int target, vector<bool>& visited){
        if(target == 0){
            ans.push_back(path);
            return ;
        }
        if(target < 0){
            return ;
        }

        for(int i=start; i<candidates.size();i++){
            if(visited[i]){
                continue;
            }
            if(i>0 && candidates[i] == candidates[i-1] && !visited[i-1]){
                continue;
            }
            visited[i] = true;
            path.push_back(candidates[i]);
            backtrack(candidates,i+1,target-candidates[i],visited);
            visited[i] = false;
            path.pop_back();
        }

    }
};
```

## 77. 组合

https://leetcode-cn.com/problems/combinations/

组合问题的套路模板。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combine(int n, int k) {
        backtrack(n,k,1);
        return ans;
    }

    void backtrack(int n, int k, int start){
        if(path.size() == k){
            ans.push_back(path);
            return ;
        }

        for(int i=start; i<=n; i++){
            path.push_back(i);
            backtrack(n,k,i+1);
            path.pop_back();
        }
    }
};
```

## 1291. 顺次数
https://leetcode-cn.com/problems/sequential-digits/

从1-9选择首位数字，每次选完了开始递归回溯继续选择，从当前选择数字的+1开始选；

如果选到的是9，就结束；

然后计算选到的和是不是在low和high之间。

这里是传递参数，所以没有做显式回溯；如果是传引用，就要做显式回溯。

```C++
class Solution {
public:
    vector<int> ans;
    vector<int> sequentialDigits(int low, int high) {
        for(int i=1;i<=9;i++){
            backtrack(i,i,low,high);
        }
        sort(ans.begin(),ans.end());
        return ans;
    }

    void backtrack(int path, int end, int low, int high ){
        if(path > high){
            return ;
        }

        if(path >= low && path <= high){
            ans.push_back(path);
        }
        if(end >= 9){
            return;
        }
        path = path*10 + end+1;
        backtrack(path, end+1,low,high);
    }
};
```