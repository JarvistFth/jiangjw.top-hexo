---
title: leetcode-贪心算法系列（1）
date: 2020-09-06 22:17:16
categories: 
- leetcode
- 贪心
tags: [leetcode ,贪心]
keywords: [leetcode,贪心]
---
局部最优解就是全局最优解，选择的时候，直观就行。

贪心问题，按照通过率排序，第一页题目。

<!---more--->
# 1282. 用户分组
https://leetcode-cn.com/problems/group-the-people-given-the-group-size-they-belong-to/

就用map每个group容量记录一个用户id的vector。如果map记录的该容量已满（记录的vector-size == groupSize(i)），就将记录的vector放入ans，然后清空该容量对应的vector。

```C++
class Solution {
public:
    unordered_map<int,vector<int>> map;
    vector<vector<int>> groupThePeople(vector<int>& groupSizes) {
        vector<vector<int>> ans;
        vector<int> group;
        for(int i=0;i<groupSizes.size();i++){
            map[groupSizes[i]].push_back(i);
            if(map[groupSizes[i]].size() == groupSizes[i]){
                ans.push_back(map[groupSizes[i]]);
                map[groupSizes[i]].clear();
            }
        }
        return ans;

    }
};
```

# 1221. 分割平衡字符串
https://leetcode-cn.com/problems/split-a-string-in-balanced-strings/

每次都取最短的LR字符串就可以了。比如有一个L，就要找一个R；不要多选。

```C++
class Solution {
public:
    int balancedStringSplit(string s) {
        int left = 0, right = 0;
        int ans = 0;
        for(auto c:s){
            if(c == 'L'){
                left++;
            }
            if(c == 'R'){
                right++;
            }
            if(left == right){
                ans++;
            }
        }
        return ans;
    }
};
```

# 763. 划分字母区间

遍历字符串，用一个map记录每个字符最后出现的下标。当遍历到的字符串下标未到已经遍历过的字符串的最大最后出现下标，说明还需要延长；到达了最大最后出现下标，就获取此时长度，更新长度计算起点。

https://leetcode-cn.com/problems/partition-labels/

```C++
class Solution {
public:
    unordered_map<char,int> map;
    vector<int> ans;
    vector<int> partitionLabels(string S) {
        if(S.empty()){
            return {};
        }

        for(int i=0;i<S.size();i++){
            map[S[i]] = i;
        }
        int l=0,r=0;
        for(int i=0;i<S.size();i++){
            r = max(r,map[S[i]]);
            if(i == r){
                ans.push_back(r-l+1);
                l = i+1;
            }
        }
        return ans;
    }
};
```

# 1111. 有效括号的嵌套深度
https://leetcode-cn.com/problems/maximum-nesting-depth-of-two-valid-parentheses-strings/

这题的描述真的感人。。。其实就是要将括号划分为两个阵营，让他们有效括号长度尽可能接近就行。。

所以我们就按照奇偶划分，奇数的括号给A，偶数的括号给B。这样他们肯定是最接近的。

```C++
class Solution {
public:
    vector<int> maxDepthAfterSplit(string seq) {
        vector<int> ans;

        int n = 0;

        for(auto c:seq){
            if(c == '('){
                n++;
                ans.push_back(n%2);
            }else{ 
                ans.push_back(n%2);
                n--;
            }
        }
        return ans;
    }
};
```

# 861. 翻转矩阵后的得分
https://leetcode-cn.com/problems/score-after-flipping-matrix/

最高位一定要是1，这样分数才会高；所以要把第一列首位全部转换成1；

换完第一列后，扫描后面每一列；因为第一列不能动，所以不能转换行，只能转换列。所以这时候判断一下一列中1多还是0多，如果0多才进行转换。

```C++
class Solution {
public:
    int matrixScore(vector<vector<int>>& A) {
        if(A.size() == 0){
            return 0;
        }

        //第一列转成1
        for(int row = 0; row<A.size(); row++){
            if(A[row][0] == 0) {
                for(int col = 0; col<A[row].size(); col++){
                    A[row][col] = 1 - A[row][col];
                }
            }
        }
        //其他列，0多的转成1
        for(int col=0;col<A[0].size();col++){
            int countOne = 0;
            for(int row=0;row<A.size();row++){
                if(A[row][col] == 1){
                    countOne++;
                }
            }

            if(countOne <= A.size()/2){
                for(int row=0;row<A.size();row++){
                    A[row][col] = 1 - A[row][col];
                }
            }
        }

        int ans = 0;

        for(int row=0;row<A.size();row++){
            int rowSum = 0;
            for(int col=0;col<A[row].size();col++){
                rowSum += A[row][col] * pow(2,A[row].size()- col - 1);
            }
            ans += rowSum;
        }
        return ans;


    }
};
```

# 921. 使括号有效的最少添加
https://leetcode-cn.com/problems/minimum-add-to-make-parentheses-valid/

这里右括号先出现，也要添加左括号才能匹配。所以我们对左括号做记录，遇到一个右括号的时候，看看左括号计数是否为0。

如果左括号此时为0，证明当前右括号无法匹配，需要添加；否则的话前面有左括号匹配，让待匹配左括号计数-1.

最后要求的答案就是无法匹配的右括号计数和待匹配左括号的计数之和。

```C++
class Solution {
public:
    int minAddToMakeValid(string S) {
        if(S.empty()){
            return 0;
        }

        int right = 0;
        int left=0;

        for(auto c:S){
            if(c == '('){
                left++;
            }
            else if(c == ')'){
                if(left > 0){
                    left--;
                }
                else{
                    right++;
                }
            }
        }
        return right+left;
    }
};
```

# 944. 删列造序
https://leetcode-cn.com/problems/delete-columns-to-make-sorted/

直接暴力看每一列是不是有序的，如果是，就不用删除；否则要删除。

```C++
class Solution {
public:
    int minDeletionSize(vector<string>& A) {

        if(A.empty()){
            return 0;
        }

        int ans = 0;
        // vector<int> d;

        for(int j=0;j<A[0].size();j++){
            for(int i=0;i<A.size();i++){
                if(i>0 && A[i][j] < A[i-1][j]){
                    // d.push_back(j);
                    ans++;
                    break;
                }
            }
        }
        return ans;

    }
};
```

# 1403. 非递增顺序的最小子序列
https://leetcode-cn.com/problems/minimum-subsequence-in-non-increasing-order/

先排序，然后从最大的开始选起，选到选择的元素和大于剩下的元素和为止。

```C++
class Solution {
public:
    vector<int> minSubsequence(vector<int>& nums) {    
        if(nums.empty()){
            return {};
        }
        vector<int> ans;
        sort(nums.begin(),nums.end());
        int sum = 0;
        for(auto i:nums){
            sum+=i;
        }
        int r = 0,l=0;
        for(int i=nums.size()-1;i>=0;i--){
            r+=nums[i];
            ans.push_back(nums[i]);
            l = sum-r;
            if(r>l){
                break;
            }
        }
        return ans;
    }
};
```

# 1217. 玩筹码
https://leetcode-cn.com/problems/minimum-cost-to-move-chips-to-the-same-position/

移动两位不需要成本，所以奇数位置的全部挪到一起，偶数位置的挪到一起；然后两坨谁的数量少，就挪谁。

```C++
class Solution {
public:
    int minCostToMoveChips(vector<int>& position) {
        if(position.empty()){
            return 0;
        }
        int odd=0,even=0;
        for(auto c:position){
            if(c % 2 == 0){
                even++;
            }else{
                odd++;
            }
        }
        return min(odd,even);
    }
};
```