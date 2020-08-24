---
title: double-pointer-5/6
date: 2020-08-24 23:02:32
tags: [leetcode,双指针]
categories: 
- leetcode
- 双指针
---
LeetCode双指针，第5第6页题目。
<!---more--->


# 977. 有序数组的平方
https://leetcode-cn.com/problems/squares-of-a-sorted-array/

左右下标遍历数组，哪个大，拿到哪边的数字的平方，同时移动哪边的指针。

先设定好ans数组和A同样大小，然后用一个变量记录填充的下标，因为我们拿到的是从大到小的元素，所以这个下标从A.size()-1开始。每填充一个元素，下标-1 。

```C++
class Solution {
public:
    vector<int> sortedSquares(vector<int>& A) {
        vector<int>ans (A.size());
        int left=0,right=A.size()-1;
        int pos = A.size()-1;

        while(left<=right){
            int l = A[left] * A[left];
            int r = A[right] * A[right];


            if(l<r){
                ans[pos] = r; 
                right--;
            }else{
                ans[pos] = l;
                left++;
            }
            pos--;
        }
        return ans;

    }
};
```

# 141. 环形链表

labuladong大佬算法提供的模板。因为链表是有环的，所以让快指针每次比慢指针多走1步，当慢指针走完链表的时候，快指针应该和慢指针会重合，所以就是环形链表。

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
    bool hasCycle(ListNode *head) {
        if(head == NULL){
            return false;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast != NULL && fast->next != NULL){
            fast = fast->next->next;
            slow = slow->next;
            if(fast == slow){
                return true;
            }
        }
        return false;
    }
};
```

# 167. 两数之和 II - 输入有序数组
双指针，因为是有序数组，所以我们从小到大遍历。遇到左右指针相加==target的就返回；小于target就让left++，大于target让right++。

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left=0,right=numbers.size()-1;
        vector<int> ans(2);

        while(left<right){
            if(numbers[left] + numbers[right] == target){
                ans[0] = left+1;
                ans[1] = right+1;
                break;
            }else if(numbers[left] + numbers[right] > target){
                right--;
            }else{
                left++;
            }
        }

        return ans;

    }
};
```

# 61. 旋转链表
https://leetcode-cn.com/problems/rotate-list/

先把链表连成环，然后计算出旋转后头节点的位置，再拿到头节点的前驱节点，让前驱节点的后继节点指向null，更新头节点。

旋转后头节点的位置就是length - k%length；所以前驱节点的位置是length - k%length - 1。

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
    ListNode* rotateRight(ListNode* head, int k) {
        if(head == NULL){
            return NULL;
        }

        if(k == 0){
            return head;
        }

        int length = 1;

        ListNode* cur = head;

        while(cur->next != NULL){
            cur = cur->next;
            length++;
        }
        
        cur->next = head;
        cur = head;

        int pos = length - k%length - 1;

        while(pos>0){
            cur = cur->next;
            pos--;
        }

        head = cur->next;
        cur->next = NULL;
        return head;


    }
};
```

