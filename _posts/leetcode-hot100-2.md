---
title: leetcode-hot100-2
date: 2020-09-22 19:13:30
categories:
- leetcode
- hot100
tags: [leetcode ,hot100]
keywords: [leetcode,hot100]
---
leetcode hot100 合集，第二部分。
<!---more--->

# 70. 爬楼梯
https://leetcode-cn.com/problems/climbing-stairs/

经典dp。dp[n]表示到n层的爬法个数；到第i层的爬法，我们可以从i-1层爬1层，或者从i-2层爬2层。所以dp[i]=dp[i-1]+dp[i-2]。

初始状况dp[1]=1,dp[2]=2 。dp[n]是所求。

```C++
class Solution {
public:
    int times = 0;
    int climbStairs(int n) {
        
        if(n<=2){
            return n;
        }
        int *dp = new int [n+1];
        dp[1] = 1;
        dp[2] = 2;
        for(int i=3;i<=n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
};
```

# 72. 编辑距离
https://leetcode-cn.com/problems/edit-distance/

知道是dp。。但是不会构造dp。。评论的题解很棒。。。嗯。。。

dp[i][j]表示word1[0...i]变成word2[0...j]的最小操作数。

考虑：word1[0...i] word2[0...j]和word1[i],word2[j]；

1. 如果word1[i]==word2[j]，操作数是dp[i-1][j-1]；因为当前字符相同，不用操作；
2. 否则需要操作。操作有三种，改变、增加、删除；
3. 如果是改变，就是认为word1[0...i-1] == word2[0...j-1]，dp[i-1][j-1]的操作数上+1，将word1[i]变成word2[j]；
4. 如果是删除，就是认为word1[0...i-1] == word2[0...j]，需要在word1[0...i]上删除word1[i]，也是一步操作。而word1[0...i-1]->word2[0...j] = dp[i-1][j]。
5. 同理，如果是增加，就是认为word1[0...i] == word2[0...j-1],需要在word1[0...j-1]上增加一个word2[j]，也是一步操作。而word1[0...i]->word2[0...j-1] = dp[i-1][j]。

综上：dp[i][j] = 1+min({dp[i-1][j-1],dp[i-1][j],dp[i][j-1]})。

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