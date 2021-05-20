---
title: 动态规划2-字符串dp
date: 2020-08-26 23:50:46
tags: [leetcode,动态规划]
categories: 
- leetcode
- 动态规划
---
LeetCode字符串的动态规划。

字符串的dp一般都是二维，i和j分别指向当前的两个字符串的某一个字符。

<!---more--->

## 72. 编辑距离
https://leetcode-cn.com/problems/edit-distance/

dp[i][j]为w1[:i-1]和w2[:j-1]的编辑距离；
所以如果w1[i-1]==w2[j-1]，dp[i][j] = dp[i-1][j-1]，相同的就不用再编辑了；
否则，就看是要增加/删除/修改（对w1来说）；分别对应的case为dp[i-1][j]（在w2的j处增加了一个相同的字符，也就是在w1处删掉一个字符）/dp[i][j-1]（在w1的i处增加了一个字符）/dp[i-1][j-1]；取三者最小值。

base case：dp[i][0]和dp[0][j]，代表一个字符为空的情况下，要怎么编辑才能让二者相等，所以编辑次数就是另一个字符串当前遍历的长度。

```C++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size();
        int n = word2.size();

        vector<vector<int>> dp(m+1,vector<int>(n+1));

        for(int i=0;i<=m;i++){
            dp[i][0] = i;
        }
        for(int j=0;j<=n;j++){
            dp[0][j] = j;
        }

        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(word1[i-1] == word2[j-1]){
                    dp[i][j] = dp[i-1][j-1];
                }else{
                    dp[i][j] = 1+min({dp[i-1][j-1],dp[i-1][j],dp[i][j-1]});
                }
            }
        }
        return dp[m][n];
    }
};
```

## 97. 交错字符串
https://leetcode-cn.com/problems/interleaving-string/

dp[i,j]表示s1[0,i-1],s2[0,j-1]能不能组成s3[i+j-1]的交错字符串； 

如果前一个位置不是交错字符串，直接就是false；否则的话，还要判断一下s3[i+j-1]和s1[i]/s2[j]的字符是否相等；

basecase表示某一个字符串不取字符时的情况，其实就是判断s1/s2和s3是否相等。

```C++
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        int n1 = s1.size(), n2 = s2.size(), n3=s3.size();
        if(n1+n2 < n3){
            return false;
        }
        vector<vector<bool>> dp(n1+1,vector<bool>(n2+1,false));

        dp[0][0] = true;

        for(int i=1; i<=n1; i++){
            dp[i][0] = dp[i-1][0] && s1[i-1] == s3[i-1];
        }
        for(int j=1; j<=n2; j++){
            dp[0][j] = dp[0][j-1] && s2[j-1] == s3[j-1];
        }


        for(int i=1; i<=n1; i++){
            for(int j=1; j<=n2; j++){
                dp[i][j] = (dp[i-1][j] && s1[i-1] == s3[i-1+j]) || (dp[i][j-1] && s2[j-1] == s3[i+j-1]);
            }
        }

        return dp[n1][n2];
    }
};

```

## 139. 单词拆分
https://leetcode-cn.com/problems/word-break/

令dp[i]为以i结尾的单词能否被Dict里面的单词拆分；

则dp[i] = dp[j] && Dict.contain(s.substr(j,i-j)); 将单词从s[:i]分成s[:j]和s[j+1:i]两段，看这两段是否都在dict里面。j是从[0,i)进行枚举。

```C++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool> dp(s.size()+1,false);
        unordered_set<string> dict(wordDict.begin(),wordDict.end());

        dp[0] = true;

        for(int i=1;i<=s.size();i++){
            for(int j=0;j<i;j++){
                if(dp[j] && dict.count(s.substr(j,i-j))){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```

## 516. 最长回文子序列
https://leetcode-cn.com/problems/longest-palindromic-subsequence/

dp[i][j]表示s[i...j]最长的回文子序列。也就是从末尾和开头遍历字符串做比较，如果相等，就选择i和n-j添加到回文子序列中，dp[i][j] = max(dp[i][j],dp[i-1][j-1])；

如果不相等，就不取i或者不取j，所以dp[i][j] = max(dp[i-1][j],dp[i][j-1])

```C++
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        int n = s.size();
        vector<vector<int>> dp(n+1,vector<int>(n+1));//dp[i][j]: s[i...j]'s max len of palindrome

        for(int i=1; i<=n; i++){
            for(int j=1; j<=n; j++){
                if(s[i-1] == s[n-j]){
                    dp[i][j] = max(dp[i][j],dp[i-1][j-1]+1);
                }else{
                    dp[i][j] = max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }

        return dp[n][n];
    }
};
```

## 10. 正则表达式匹配
https://leetcode-cn.com/problems/regular-expression-matching/

匹配分两种情况：1.匹配一个字符；2.匹配多个字符；

设当前正则字符为p，待匹配字符为s；

1. 匹配一个字符，s == p || p == '.'，然后匹配下一个字符s+1和p+1;
2. 匹配多个字符，p+1 = '*'。 
    - 然后考虑匹配0个和匹配多个。匹配0个下一个待匹配的仍然是是s，正则字符p和p+1匹配完了，所以是p+2；
    - 匹配多个：当前匹配一个字符，然后看s+1和当前待匹配的p；

```C++
class Solution {
public:
    bool isMatch(string s, string p) {
        return isMatch(s.c_str(),p.c_str());
    }

    bool isMatch(const char* s, const char* p){
        if(*p == 0){
            return *s == 0;
        }

        bool firstMatch = *s!=0 && ((*s==*p) || (*p == '.'));

        if(*(p+1) == '*'){
            return isMatch(s,p+2) || (firstMatch && isMatch(++s,p));
        }else{
            return firstMatch && isMatch(++s,++p);
        }
    }
};
```

## 32. 最长有效括号
https://leetcode-cn.com/problems/longest-valid-parentheses/

dp[i]表示，i位置处最长括号长度；此时s[i]肯定为')'。
然后分两种情况：

1. s[i-1] == '('：这种情况很好说明，s[i-1]和s[i]就构成一个有效括号对，看s[i-2]处最长有效括号长度，即dp[i-2]；所以dp[i] = dp[i-2]+2；

2. s[i-1] == ')'：这个时候，s[i-1]前一定有一个'('对应；这个位置可能是i - dp[i-1] - 1（试想一下' (()) '，意思是当前位置 - 上一个右括号对应的有效括号长度 的前一个位置的括号）。

    如果这个位置是')'，其实就是我们之前求过的值（我们只处理当前位置是右括号的情况）；
    如果是'('，那么我们之前没处理过。

    这时候dp[i] = dp[i-1] + dp[i - dp[i-1] - 2] + 2；

    dp[i-1]对应前一个括号为')'的最长长度，dp[i - dp[i-1] - 2]意味着是'('的再前一个位置的最长有效括号长度，这时候无论那个位置是左括号还是右括号，我们都已经计算过了；最后再加上当前位置是右括号的2个括号长度即可。

```C++
class Solution {
public:
    int longestValidParentheses(string s) {
        if(s.empty()){
            return 0;
        }
        int n = s.size();
        vector<int> dp(n);
        int ans = 0;

        for(int i=1; i<n; i++){
            if(s[i] == ')'){
                if(s[i-1] == '('){
                    dp[i] = i>=2?dp[i-2]+2:2;
                }else if(i-dp[i-1]>=1 && s[i-dp[i-1]-1] == '('){
                    if(i-dp[i-1]-2>=0){
                        dp[i] = dp[i-1] + dp[i-dp[i-1]-2] + 2;
                    }else{
                        dp[i] = dp[i-1] + 2;
                    }
                }
            }
            ans = max(ans,dp[i]);
        }
        return ans;
    }
};
```

## 300. 最长递增子序列
https://leetcode-cn.com/problems/longest-increasing-subsequence/

这题很经典也很简单。dp[i]代表以i结尾处最长子序列长度。答案就是dp数组的最大值。

那么dp[i] = max(dp[i],dp[j] + 1)； 当且仅当nums[j] > nums[i]的情况下。
因为是子序列，所以j从[0,i)进行枚举。然后看选不选j，选了j，dp[i] = dp[j]+1；如果不选
，就是dp[i]。选不选看当前dp[i]和dp[j]+1谁大。

注意dp数组basecase全部为1。递增子序列长度最少包括它自己。

```C++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }
        int n = nums.size();
        vector<int> dp(n,1);
        int ans = 0;
        for(int i=0; i<n; i++){
            for(int j=0; j<i ;j++){
                if(nums[i] > nums[j]){
                    dp[i] = max(dp[i],dp[j]+1);
                }
            }
            ans = max(ans,dp[i]);
        }
        return ans;
        
    }
};
```

## 354. 俄罗斯套娃信封问题
https://leetcode-cn.com/problems/russian-doll-envelopes/

简单版做法，和最长上升子序列一样。只不过这里的子序列是信封。

对每个信封先做个升序排列，然后按照最长上升子序列的方法做就可以了。

```C++
class Solution {
public:
    int maxEnvelopes(vector<vector<int>>& envelopes) {
        sort(envelopes.begin(),envelopes.end(),[](auto& e1, auto& e2){
            return e1[0] == e2[0]?e1[1] < e2[1] : e1[0]<e2[0];
        });
        int n = envelopes.size();
        vector<int> dp(n,1);
        int ans = 0;

        for(int i=0; i<n; i++){
            for(int j=0; j<i ;j++){
                if(envelopes[i][0] > envelopes[j][0] && envelopes[i][1] > envelopes[j][1]){
                    dp[i] = max(dp[i],dp[j]+1);
                }
            }
            ans = max(ans,dp[i]);
        }
        return ans;
    }
};
```