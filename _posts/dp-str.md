---
title: 动态规划-字符串dp
date: 2020-08-26 23:50:46
tags: [leetcode,动态规划]
categories: 
- leetcode
- 动态规划
---
LeetCode字符串的动态规划。

字符串的dp一般都是二维，i和j分别指向当前的两个字符串的某一个字符。

<!---more--->

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