---
title: leetcode-daily
date: 2020-09-22 12:24:09
categories:
- leetcode
- 杂题
tags: [leetcode ,杂题, 每日一题]
keywords: [leetcode,杂题, 每日一题]
---
一些不好归类的杂题和每日一题。
<!---more--->
## 50. Pow(x, n)
https://leetcode-cn.com/problems/powx-n/

经典快速幂。递归版：

```C++
class Solution {
public:
    double myPow(double x, int n) {

        if(n == 0){
            return 1;
        }
        else if(n%2 == 1){
            return (myPow(x,n-1)*x);
        }else if(n%2 == -1){
            return 1/(myPow(x,-(n+1))*x);
        }else{
            double tmp = myPow(x,n/2);
            return tmp*tmp;
        }
    }
};
```

非递归版：
```C++
class Solution {
public:
    double myPow(double x, int n) {

        // if(n == 0){
        //     return 1;
        // }
        // else if(n%2 == 1){
        //     return (myPow(x,n-1)*x);
        // }else if(n%2 == -1){
        //     return 1/(myPow(x,-(n+1))*x);
        // }else{
        //     double tmp = myPow(x,n/2);
        //     return tmp*tmp;
        // }
        double ans = 1.0;
        long long N = n;
        if(n<0){
            N = -1*(long long)n;
        }

        while(N){
            if(N&1){
                ans = ans * x;
            }
            x *= x;
            N>>=1;
        }
        return n<0?1.0/ans:ans;
    }
};
```