---
title: double-pointer-3/4
date: 2020-08-15 20:23:20
tags: [leetcode,双指针]
categories: leetcode-doublePointer
---
LeetCode双指针，第3第4页题目。
<!---more--->

# 283. 移动零
https://leetcode-cn.com/problems/move-zeroes/

双指针，一个遍历数组，一个保存非零元素下标。将非零元素全部往前移，最后将后面所有元素置为0.

```C++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int i=0,j=0;

        for(int i=0;i<nums.size();i++){
            if(nums[i] != 0){
                nums[j] = nums[i];
                j++;
            }
        }
        while(j<nums.size()){
            nums[j++] = 0;
        }
    }
};
```

# 27. 移除元素
https://leetcode-cn.com/problems/remove-element/

双指针，一个遍历数组，一个记录非删除元素的下标。遇到不是删除的元素，就将非删除元素目前的下标的位置赋值。

返回长度就是当前的非删除元素下标+1 。

```C++
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        if(nums.size() == 0){
            return 0;
        }
        int i=0,j=0;
        for(int i=0;i<nums.size();i++){
            if(nums[i] != val){
                nums[j] = nums[i];
                j++;
            }
        }
        return j;
        
    }
};
```

# 剑指 Offer 04. 二维数组中的查找
https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/

这类有序矩阵问题都是通过查找四个角的元素，找到其左/右、上/下的规律性。如果这个角的元素等于target，就返回；否则就对应的往前一列/后一列或者上一行/下一行去搜索。

```C++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        if(matrix.size() == 0){
            return false;
        }

        int i=0,j=matrix[0].size()-1;

        while(i<matrix.size() && j>=0){
            if(matrix[i][j] == target){
                return true;
            }else if(matrix[i][j] < target){
                i++;
            }else{
                j--;
            }
        }
        return false;
    }
};
```

# 826. 安排工作以达到最大收益
https://leetcode-cn.com/problems/most-profit-assigning-work/

将工作结合，然后按大到小排序。双指针遍历，一个遍历worker，一个遍历job。因为已经排好序，所以用贪心思想，碰到worker第一个可以做的工作，就将job的profit加入到答案中。

```C++
class Solution {
public:
    int maxProfitAssignment(vector<int>& difficulty, vector<int>& profit, vector<int>& worker) {
        if(worker.size() == 0 || difficulty.size() == 0){
            return 0;
        }
        int size = difficulty.size();
        vector<pair<int,int>> job(size);
        for(int i=0;i<job.size();i++){
            job[i].first = difficulty[i];
            job[i].second = profit[i];
        }
        sort(job.begin(),job.end(),[](const pair<int,int>&a, const pair<int,int>&b){
            return a.second>b.second;
        });
        sort(worker.begin(),worker.end(),[](int a,int b){
            return a>b;
        });

        int ans = 0;
        int j=0;
        for(int i=0;i<worker.size();i++){
            while(j<size){
                if(worker[i] >= job[j].first){
                    ans += job[j].second;
                    break;
                }
                j++;
            }
        }
        return ans;
    }

   
};
```

# 面试题 16.06. 最小差
https://leetcode-cn.com/problems/smallest-difference-lcci/

最小差是与当前数组元素最接近的值的差值。

所以先对ab排序，然后用一个指针遍历b数组，在a数组中进行二分查找，找到第一个大于等于b[j]的值。如果找到，就和该值的前一个下标位置的值作比较，看他们和b[j]的值谁大；否则的话差值就是头一个元素或者末尾元素与b[j]的差值。

最后再将当前差值和之前保存的最小差值作比较取最小值就可以了。

```C++
class Solution {
public:
    int smallestDifference(vector<int>& a, vector<int>& b) {
        sort(a.begin(),a.end());
        sort(b.begin(),b.end());

        long ans = LONG_MAX;
        int i=0,j=0;
        long difference = -1;

        for(j;j<b.size();j++){
            auto index = lower_bound(a.begin(),a.end(),b[j]);
                if(index == a.end()){
                    difference = abs((long)b[j] - a.back());
                }

                else if(index == a.begin()){
                    difference = abs((long)b[j] - a.front());
                }
                else{
                    auto previous = index - 1;
                    difference = min(abs((long)b[j] - *previous),abs((long)b[j] - *index));
                }
                ans = min(ans,difference);            
        }
        return ans;
    }
};
```

# 287. 寻找重复数
https://leetcode-cn.com/problems/find-the-duplicate-number/

思想类似链表判断是否有环。这里利用了数组里面数字是1-n的特点，让数组代替链表中的next，让快慢指针通过数组下标来进行行进。

```C++
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int slow=0, fast = 0;

        while(true){
            fast = nums[nums[fast]];
            slow = nums[slow];
            if(fast == slow){
                break;
            }
        }
        slow = 0;
        while(slow != fast){
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    }
};
```
# 75. 颜色分类
https://leetcode-cn.com/problems/sort-colors/

这题和之前一道移动0有点像。想法是将0移动到头，将2移动到末尾，剩下的就是1了。

注意的是如果当前是0，可以直接交换然后i++，因为交换完后i的左侧已经有序；但是如果当前是2的时候，交换完了nums[i]可能还是0，所以这时候i不可以++ 。

```C++
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int i=0, l=0,r=nums.size()-1;

        while(i<=r){
            if(nums[i] == 0){
                nums[i] = nums[l];
                nums[l] = 0;
                l++;i++;
            }else if(nums[i] == 1){
                i++;
            }
            else if(i<=r && nums[i] == 2){
                nums[i] = nums[r];
                nums[r] = 2;
                r--;
            }
        }
    }

    
};
```