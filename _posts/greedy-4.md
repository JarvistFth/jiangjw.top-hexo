---
title: greedy-4
date: 2020-09-11 19:59:25
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---

贪心问题，按照通过率排序，第四页题目。

<!---more--->

# 1414. 和为 K 的最少斐波那契数字数目
https://leetcode-cn.com/problems/find-the-minimum-number-of-fibonacci-numbers-whose-sum-is-k/

求出最大为k的斐波那契数列；然后从最大的取起，取到k为0为止。

```C++
class Solution {
public:
    int findMinFibonacciNumbers(int k) {
        int ans = 0;
        int i=2;
        vector<int> dp(2,1);
        
        while(1){
            dp.push_back(*dp.rbegin() + *(dp.rbegin()+1));
            if(dp.back() > k){
                break;
            }
        }

        for(int i=dp.size()-1;i>=0;i--){
            if(k >= dp[i]){
                k-=dp[i];
                ans++;
            }
            if(k == 0){
                break;
            }
        }
        return ans;
    }
};
```

# 860. 柠檬水找零
https://leetcode-cn.com/problems/lemonade-change/

枚举就完事了，枚举5块、10块、20块的数量。注意找零的时候，20块可以找1张10块1张5块，也可以找3张5块。如果5块的钱数小于0，证明没法找。

```C++
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int five=0,ten=0,twenty=0;

        for(auto b:bills){
            if(b == 5){
                five++;
            }else if(b == 10){
                ten++;
                five--;
            }else if(b == 20){
                twenty++;
                if(ten>0){
                    ten--;five--;
                }else{
                    five-=3;
                }
            }
            if(five < 0){
                return false;
            }
        }
        return true;
    }
};
```

# 455. 分发饼干
https://leetcode-cn.com/problems/assign-cookies/comments/

贪心就是先将两个数组排序，然后从最小的饼干数组开始取起。

然后双指针控制比较，
当g[i]<=s[j]时，i++，j++；否则j++（排序保证后面的比前面的大，所以此时如果s[j]< g[i]，后面也取不到这个j了）。

```C++
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        int ans = 0;
        if(g.empty() || s.empty()){
            return 0;
        }

        sort(g.begin(),g.end(),less<>());
        sort(s.begin(),s.end(),less<>());
        int i=0,j=0;
        while(i<g.size() && j<s.size()){
            if(g[i] <= s[j]){
                i++;
                ans++;
            }
            j++;
        }
        return ans;
    }
};
```

# 134. 加油站
https://leetcode-cn.com/problems/gas-station/submissions/

首先判断总的油量是不是大于花费油量，是的话才能走完全程一个环；

然后判断当前油量，是否大于0，如果不是，证明当前坐标不能作为起点。

```C++
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        if(gas.empty() || cost.empty()){
            return -1;
        }

        int sum = 0, currentGas = 0;
        int pos = 0;
        
        for(int i=0;i<gas.size();i++){
            sum += gas[i] - cost[i];
            currentGas += gas[i] - cost[i];
            if(currentGas < 0){
                pos = i+1;
                currentGas = 0;
            }
        }
        return sum<0? -1:pos;
    }
};
```

# 1005. K 次取反后最大化的数组和
https://leetcode-cn.com/problems/maximize-sum-of-array-after-k-negations/

排序，让前K个负数取反；

如果负数不足K个，还需要对最靠近0的元素继续进行取反。所以维护一个最小值记录最靠近0的元素，如果K>0并且K是奇数，说明还需要对这个元素进行一次反转；原来的和是加了这个值，所以在原来的和的基础上减去2倍的最小值就可以了。

```C++
class Solution {
public:
    int largestSumAfterKNegations(vector<int>& A, int K) {
        if(A.empty()){
            return 0;
        }

        sort(A.begin(),A.end());
        int ans = 0;
        int minval = INT_MAX;
        for(auto& a:A){
            if(a < 0 && K>0){
                a = (-1) * a;
                K--;
            }

            ans += a;
            minval = min(minval,a);
        }

        if(K > 0 && (K%2 == 1)){
            ans = ans - 2*minval;
        }
        return ans;

    }
};
```

# 1090. 受标签影响的最大值
https://leetcode-cn.com/problems/largest-values-from-labels/

将label和value绑定在一起，可以用struct，也可以用pair；然后按照value的价值进行排序。

每次取的时候，都取value最大的；然后统计他的label，以及集合总数。

当集合总数超过num_wanted时，break循环；

如果要加入的label是大于uselimit的，就跳过这个选项；

这里每个uselimit是针对每个label的，所以需要记录每个label用到的次数；用一个unordered_map记录。

```C++
class Solution {
public:
    int largestValsFromLabels(vector<int>& values, vector<int>& labels, int num_wanted, int use_limit) {
        if(values.empty() || labels.empty()){
            return 0;
        }

        vector<pair<int,int>> objects;
        unordered_map<int,int> labelUsed;
        int ans = 0;
        int count = 0;
        int i=0,j=0;
        while(i<values.size() && j<values.size()){
            objects.emplace_back(labels[i],values[i]);
            i++;j++;
        }

        sort(objects.begin(),objects.end(),[](auto& a, auto& b){
            return a.second > b.second;
        });

        for(auto& o:objects){
            labelUsed[o.first]++;
            if(labelUsed[o.first] <= use_limit){
                ans += o.second;
                count++;
            }
            if(count >= num_wanted){
                break;
            }
        }
        return ans;

    }
};
```

# 621. 任务调度器
https://leetcode-cn.com/problems/task-scheduler/

每个相同任务之间要间隔n；所以我们直觉地将任务次数最多的任务放在一轮处理的开头。这样每一轮中间可以插入其他任务或者空转时间。

这时候除去末尾最后一轮任务的总时间就是(maxCount - 1) * (n + 1);

然后我们再看看所有的任务的次数，只要有次数是最大值的，我们就把它添加到最后一轮的时间里面。

另外需要注意n=0的特殊情况，这时候我们返回task.size()就可以了。

```C++
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        unordered_map<char,int> map;

        int maxCount = 0;
        for(auto c:tasks){
            map[c]++;
            maxCount = max(maxCount,map[c]);
        }

        int ans = (maxCount - 1) * (n + 1);

        for(auto& m:map){
            if(m.second == maxCount){
                ans++;
            }
        }

        return max((int)tasks.size(),ans);
    }
};
```



# 991. 坏了的计算器
https://leetcode-cn.com/problems/broken-calculator/

让Y向X靠近。如果X比Y大，只能递减，步骤数就是X-Y；

否则的话，如果Y是偶数让Y/2，操作数是1；如果Y是奇数，证明X需要-1才等于Y；让Y=(Y+1)/2，操作数是2。

```C++
class Solution {
public:
    int brokenCalc(int X, int Y) {
        int ans = 0;
        while(Y > X){
                Y = Y/2;
            if(Y % 2 == 0){
                ans++;
            }else{
                Y = (Y+1)/2;
                ans+=2;
            }
        }

        return ans + X - Y;

    }
};
```