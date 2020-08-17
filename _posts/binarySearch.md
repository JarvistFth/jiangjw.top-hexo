---
title: binarySearch
date: 2020-07-29 23:43:11
categories: leetcode-binarySearch
tags: [leetcode ,二分查找]
keywords: [leetcode,二分查找]
---
二分查找算法。
<!---more--->

## 704. 二分查找
https://leetcode-cn.com/problems/binary-search/

有序数组的二分查找，最基本的二分查找框架。
左右index设计，right=nums.size()-1的时候，left<=right；如果是nums.size()，是left<right 。

当left==right+1的时候，跳出循环；

如果是left<right的话，left==right的时候跳出循环，这时候还有left==right的位置的元素还没被处理。

```C++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0, right = nums.size()-1;
        while(left<=right){
            int mid = left + (right - left)/2;
            if(nums[mid] < target){
                left = mid+1;
            }else if(nums[mid] > target){
                right = mid - 1;
            }else if(nums[mid] == target){
                return mid;
            }
        }
        return -1;
    }
};
```


左右边界的二分查找：
左边界；
```C++
public:
    int search(vector<int>& nums, int target) {
        int left = 0, right = nums.size()-1;
        while(left<=right){
            int mid = left + (right - left)/2;
            if(nums[mid] < target){
                left = mid+1;
            }else if(nums[mid] > target){
                right = mid - 1;
            }else if(nums[mid] == target){
                return mid-1;
            }
        }
        return left>=nums.size() || nums[left] != target?left:-1;
    }
};

```
有边界就是return right；检查条件改成right<0 。


## 剑指 Offer 53 - II. 0～n-1中缺失的数字
https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/

这里就是检查左边界的应用。

```C++
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int left=0,right = nums.size()-1;
        while(left <= right){
            int mid = left + (right - left)/2;
            if(nums[mid] > mid){
                right = mid;
            }else if(nums[mid] == mid){
                left = mid+1;
            }
        }
        return left == nums.size()?left+1:left;

    }
};
```


## 441. 排列硬币
https://leetcode-cn.com/problems/arranging-coins/

等差数列求和，如果某一行应该的总和比实际的大，那么这一行就没有排满。检查左边界；如果left>right时跳出循环，这时候right是所求解（right是不符合条件的边界）。

```C++
class Solution {
public:
    int arrangeCoins(int n) {
        int left = 1;
        int right = n;
        long mid = 0;
        while(left <= right){
            mid = left + (right - left)/2;
            long sum = mid *(mid+1)/2;
            if(sum > n){
                //need sum > n:
                right = mid-1;
            }else if(sum <= n){
                left = mid+1;
            }
        }
        return right;
    }
};
```


## 378. 有序矩阵中第K小的元素

## 744. 寻找比目标字母大的最小字母
https://leetcode-cn.com/problems/find-smallest-letter-greater-than-target/

有序字符数组，右边界。

```C++
class Solution {
public:
    char nextGreatestLetter(vector<char>& letters, char target) {
        int left = 0, right = letters.size()-1;
        while(left<=right){
            int mid = left + (right - left) / 2;
            if(letters[mid] > target){
                right = mid - 1;
            }else if(letters[mid] <= target){
                left = mid + 1;
            }
        }
        return left == letters.size()? letters[0]:letters[left];
    }
};
```

## 475. 供暖器
https://leetcode-cn.com/problems/heaters/

在第一个数组找第二个数组中离当前元素最近的位置，判断它在元素的左边还是右边；用heaters数组的元素减去当前的元素作为距离，返回最小的。

```C++
class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        sort(heaters.begin(),heaters.end());
        int ans = 0;
        for(auto house:houses){
            int left = 0, right = heaters.size()-1;
            while(left<=right){
                int mid = left + (right - left)/2;
                if(heaters[mid]<house){
                    left = mid + 1;
                }else if(heaters[mid] >= house){
                    right = mid - 1;
                }
            }
            int dist1 = (left == heaters.size())?INT_MAX:heaters[left] - house;
            int dist2 = (left == 0)?INT_MAX:house-heaters[right];
            ans = max(ans,min(dist1,dist2));

        }
        return ans;

    }
```
## 278. 第一个错误的版本
https://leetcode-cn.com/problems/first-bad-version/

左边界，如果碰到是错误的版本，右边界等于当前index；如果是正确的版本，左边界等于当前index+1 。最后左边界和右边界相等时返回这时候左右index对应的version都是错误的。

```C++

class Solution {
public:
    int firstBadVersion(int n) {
        long long l = 0, r = n;
        while(l<r){
            long long mid = l+(r-l)/2;
            if(isBadVersion(mid)){
                r = mid;
            }else{
                l = mid + 1;
            }
        }
        return r;


    }
};
```

## 69. x 的平方根
https://leetcode-cn.com/problems/sqrtx/
二分查找，如果当前的平方小于x，l+1;否则有边界r=mid。

返回时要注意，l是可能>=x的，如果是大于x，向下取整也就是l-1，否则就是l。

```C++
class Solution {
public:
    int mySqrt(int x) {
        long l = 0;
        long r = x/2+1;
        while(l<r){
            int mid = l+(r-l)/2;
            if((long long)mid*mid < x){
                l = mid + 1;
            }else{
                r = mid;
            }
        }
        return l*l>x?l-1:l;
    }
};
```

## 1351. 统计有序矩阵中的负数
https://leetcode-cn.com/problems/count-negative-numbers-in-a-sorted-matrix/
对每一行，二分右边界。每行的负数个数=每行个数-二分结束时l的位置（因为当l>r时结束循环，这时候l是小于0的，r是大于0的。）

```C++
class Solution {
public:
    int ans = 0;
    int countNegatives(vector<vector<int>>& grid) {
        for(int i = 0;i<grid.size();i++){
            int n = grid[i].size();
            int l=0,r=n-1;
            if(grid[i][r] < 0){
                while(l<=r){
                int mid = (l+r)/2;
                    if(grid[i][mid] >= 0){
                        l = mid + 1;
                    }else if(grid[i][mid]<0){
                        r = mid - 1;
                    }
                }
                ans += n-l;
            }else if(grid[i][l] < 0){
                ans += n;
            }            
        }
        return ans;
    }
};
```

## 面试题 17.08. 马戏团人塔
https://leetcode-cn.com/problems/circus-tower-lcci/

身高排序，体重最长递增子序列。

```C++
class Solution {
public:
    int bestSeqAtIndex(vector<int>& height, vector<int>& weight) {
        int n = height.size();
        vector<pair<int,int>> people(n);
        for(int i=0;i<height.size();i++){
            people[i] = {height[i],weight[i]};
        }
        sort(people.begin(),people.end(),compare);
        //store the weight after sorted according to the height
        vector<int> dp;
        for(int i=0;i<n;i++){
            //pos indexs the dp[] element that bigger than people[i].weight
            //if cant find the element, dp[i].weight is the biggest,and dp[i] is tallest after sorted.
            //if we can find the elements, dp[pos].weight is bigger than dp[i]. dp[i] is tallest,but his weight is less than someone before him. So we try to update the dp[pos], to make dp is still in order.
            //if the next tallest one is thinner than dp[i], we have to give him up .
            int pos = lower_bound(dp.begin(),dp.end(),people[i].second) - dp.begin();
            if(pos == dp.size()){
                dp.push_back(people[i].second);
            }else{
                dp[pos] = people[i].second;
            }
        }
        return dp.size();
    }

    static bool compare(pair<int,int>& a, pair<int,int>& b){
        if(a.first < b.first){
            return true;
        }
        else if(a.first == b.first){
            return a.second>b.second?true:false;
        }
        else return false;
    }
};
```

## 面试题 10.05. 稀疏数组搜索
https://leetcode-cn.com/problems/sparse-array-search-lcci/

二分查找，如果找到是空的字符串，就把对应的下表++或--。
```C++
class Solution {
public:
    int findString(vector<string>& words, string s) {
        int l=0,r=words.size()-1;
        while(l<=r){
            while(words[l] == ""){
                l++;
            }
            while(words[r] == ""){
                r--;
            }
            int mid = l+(r-l)/2;
            while((mid >= 1) && (words[mid] == "") ){
                mid--;
            }
            if(words[mid] == s){
                return mid;
            }else if(words[mid] < s){
                l = mid + 1;
            }else if(words[mid] > s){
                r = mid - 1;
            }
        }
        return -1;
    }
};
```
## 剑指 Offer 11. 旋转数组的最小数字
https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/
如果右边的比中间的小，证明最小值在右侧，l+；否则的话最小值在左侧，r=mid。遇到重复的r--；
 
这里要注意的是取r是number.size()-1，但是while判断还是l<r 。当l == r 时退出循环。这时l和r都指向最小值，所以返回就可以了。

```C++
class Solution {
public:
    int minArray(vector<int>& numbers) {
        int l=0,r=numbers.size()-1;
        while(l<r){
            int mid = l+(r-l)/2;
            if(numbers[mid] > numbers[r]){
                l = mid + 1;
            }else if(numbers[mid] < numbers[r]){
                r = mid;
            }else{
                r = r - 1;
            }
        }
        return numbers[l];
    }
};
```

## 436. 寻找右区间
https://leetcode-cn.com/problems/find-right-interval/

对每个区间的左端点进行排序；然后对每个右端点进行二分查找，如果在排序过的左端点容器里找到比右端点大的，此时肯定有右区间，左端点容器对应的index就是所求值；否则的话找不到比右端点大的，此时没有右区间。

```C++
class Solution {
public:
    vector<int> findRightInterval(vector<vector<int>>& intervals) {
        map<int,int> m;//([i][0],index)
        vector<int> ans;
        int n = intervals.size();
        for(int i=0;i<n;i++){
            m[intervals[i][0]] = i;
        }
        for(int i=0;i<n;i++){
            auto pos = m.lower_bound(intervals[i][1]);
            if(pos == m.end()){
                ans.push_back(-1);
            }else{
                ans.push_back(pos->second);
            }
        }
        return ans;
    }
};
```

## 719. 找出第 k 小的距离对
https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance/

二分+双指针。

对距离进行二分查找，如果出现的小于等于mid的距离对个数是>=k，就将右边界收窄，否则将左边界收窄。

双指针计算数组中所有距离小于等于mid的个数，遇到距离大于mid时，移动慢指针。最后总数就是快指针-慢指针。


```C++
class Solution {
public:
    int smallestDistancePair(vector<int>& nums, int k) {
        sort(nums.begin(),nums.end());
        int l=0,r=nums.back()-nums[0];
        while(l<r){
            int mid = l+(r-l)/2;
            int count = 0;
            int i=0;
            for(int j=0;j<nums.size();j++){
                while(nums[j] - nums[i] > mid){
                    i++;
                }                    
                count += j-i;

            }
            if(count>=k){
                r=mid;
            }else{
                l=mid+1;
            }
        }
        return l;

    }

   
};
```

## 剑指 Offer 53 - I. 在排序数组中查找数字 I
https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/

二分查找到target的左边界，让左边界往右移动，如果和target相等，ans++。

```C++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int ans = 0;
        int l=0,r=nums.size();
        while(l<r){
            int mid = l+(r-l)/2;
            if(nums[mid] < target){
                l=mid+1;
            }else if(nums[mid] >= target){
                r=mid;
            }
        }
        while(l<nums.size() && nums[l] == target){
            l++;
            ans++;
        }
        return ans;
    }
};
```

## 875. 爱吃香蕉的珂珂
https://leetcode-cn.com/problems/koko-eating-bananas/

二分查找速度；以这个速度吃香蕉，如果全部吃完所需时间大于H，就加快速度；否则减少速度。

```C++
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int H) {
        int l=1,r=*max_element(piles.begin(),piles.end());

        while(l<=r){
            int mid = l+(r-l)/2;
            if(int h = getEatingHours(piles,mid) <= H){
                r = mid - 1;
            }else{
                l = mid + 1;
            }
        }
        return l;
    }

    int getEatingHours(vector<int>& piles, int mid){
        int hours = 0;
        for(auto p:piles){
            hours += p/mid;
            if(p%mid){
                hours++;
            }
        }
        return hours;
    }
};
```