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
# 50. Pow(x, n)
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

# 面试题 16.19. 水域大小
https://leetcode-cn.com/problems/pond-sizes-lcci/

dfs，迷宫类题目。

```C++
class Solution {
public:
    vector<int> pondSizes(vector<vector<int>>& land) {
        vector<int> ans;
        int area = 0;
        for(int i=0;i<land.size();i++){
            for(int j=0;j<land[i].size();j++){
                if(land[i][j] == 0){
                    area = backtrack(land,i,j);
                    ans.push_back(area);
                }
            }
        }
        sort(ans.begin(),ans.end());
        return ans;
    }

    int backtrack(vector<vector<int>>& land, int x, int y){
        if(x<0 || x>=land.size() || y<0 || y>=land[x].size() || land[x][y] != 0){
            return 0;
        }

        int sum = 1;
        land[x][y] = -1;
        sum += backtrack(land,x-1,y) + backtrack(land,x-1,y-1) 
            + backtrack(land,x-1,y+1) 
            + backtrack(land,x,y-1)+ backtrack(land,x,y+1) 
            + backtrack(land,x+1,y-1) + backtrack(land,x+1,y) + backtrack(land,x+1,y+1);
        
        return sum;
    }
};
```

---
## 649. Dota2 参议院
Date:2020/12/11

https://leetcode-cn.com/problems/dota2-senate/

贪心+模拟，两个队列模拟banpick环节，每次ban的时候都找最近的那个位置对手去ban，循环该ban选过程直到某一个队列为空。

```C++
class Solution {
public:
    string predictPartyVictory(string senate) {
        queue<int> RadianQueue;
        queue<int> DireQueue;
        int n = senate.size();
        for(int i=0;i<n;i++){
            char c = senate[i];
            if(c == 'R'){
                RadianQueue.push(i);
            }else{
                DireQueue.push(i);
            }
            
        }

        while(!RadianQueue.empty() && !DireQueue.empty()){
            int rfront = RadianQueue.front();
            int dfront = DireQueue.front();
            if(rfront < dfront){
                RadianQueue.push(rfront + n);
            }else{
                DireQueue.push(dfront + n);
            }
            RadianQueue.pop();
            DireQueue.pop();
        }

        return RadianQueue.empty()?"Dire":"Radiant";
    }
};
```