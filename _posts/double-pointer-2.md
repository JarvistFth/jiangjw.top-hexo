---
title: double-pointer-3/4
date: 2020-08-15 20:23:20
tags: [leetcode,双指针]
categories: 
- leetcode
- 双指针2
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

# 接雨水
https://leetcode-cn.com/problems/trapping-rain-water/

这题还挺难理解的。。。

当然O(n^2)的方法还是比较好写的，两重循环，对于每次的height[i]找其左右的最大值。ans就是左右最大值的最小值和height[i]的差值。

链接：https://leetcode-cn.com/problems/trapping-rain-water/solution/jie-yu-shui-by-leetcode/
```C++
int trap(vector<int>& height)
{
    int ans = 0;
    int size = height.size();
    for (int i = 1; i < size - 1; i++) {
        int max_left = 0, max_right = 0;
        for (int j = i; j >= 0; j--) { //Search the left part for max bar size
            max_left = max(max_left, height[j]);
        }
        for (int j = i; j < size; j++) { //Search the right part for max bar size
            max_right = max(max_right, height[j]);
        }
        ans += min(max_left, max_right) - height[i];
    }
    return ans;
}

```

双指针法比较巧妙，O(n)的时间复杂度。主要想法是对于一个height[i]，如果我们确定了一边的较高值，那么ans就由另一边的较低值确定。所以对于height[l]<height[r]，其是由leftMaxHeight确定装雨水的量。所以如果当前height[l]是大于leftMaxHeight，就更新leftMaxHeight；否则的话这时候height[l]刚好处于凹陷的状态，ans=leftMaxHeight - height[l]。

同样的，当height[l]>=height[r]就是相反的状态。

```C++
class Solution {
public:
    int trap(vector<int>& height) {
        if(height.size() < 3){
            return 0;
        }
        int l=0,r=height.size()-1;
        int leftMaxHeight = 0, rightMaxHeight = 0;
        int ans = 0;
        while(l<r){
            if(height[l]<height[r]){
                if(height[l] > leftMaxHeight){
                    leftMaxHeight = height[l];
                }else{
                    ans+=leftMaxHeight - height[l];
                }
                l++;
            }else{
                if(height[r] > rightMaxHeight){
                    rightMaxHeight = height[r];
                }else{
                    ans+=rightMaxHeight - height[r];
                }
                r--;
            }
        }
        return ans;
    }
};
```

# 86. 分隔链表
https://leetcode-cn.com/problems/partition-list/

新建两个链表，一个存放比x小的元素，一个存放比x大的元素；然后将小的链表和大的链表合起来就可以了。
```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        if(head == NULL){
            return NULL;
        }

        ListNode* smallDummy = new ListNode(-1);
        ListNode* bigDummy = new ListNode(-1);

        ListNode* smallCur = smallDummy;
        ListNode* bigCur = bigDummy;

        while(head != NULL){
            if(head->val < x){
                smallCur->next = head;
                smallCur = smallCur->next;
            }else{
                bigCur->next = head;
                bigCur = bigCur->next;
            }
            head = head->next;
        }
        smallCur->next = bigDummy->next;
        bigCur->next = NULL;
        return smallDummy->next;
    }
};
```
原地修改的代码想法是先找到第一个比x小的元素的前一个节点p，作为插入的头节点。然后再在这个节点的后一个节点开始寻找比x小的节点，把它插入到p的后面。
原地修改版代码：
```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        if(head == NULL){
            return NULL;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;

        ListNode* p = dummy;
        ListNode* q = dummy;

        while(p->next != NULL && p->next->val<x){
            p = p->next;
        }

        //p is the insert point
        if(p->next == NULL){
            return head;
        }
        //q is the first node that are more than or equal to x
        q = p->next; 
        
        while(q->next != NULL){
            if(q->next->val < x){
                auto pnext = p->next;
                p->next = q->next;
                q->next = q->next->next;
                p->next->next = pnext;
                p = p->next;
            }else{
                q = q->next;
            }     
        }
        return dummy->next;
    }
};
```