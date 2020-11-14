---
title: leetcode-linkedlist
date: 2020-09-27 00:04:57
categories:
- leetcode
- 链表
tags: [leetcode ,链表]
keywords: [leetcode,链表]
---

一些比较经典的链表题。常用的方法包括：

1. 快慢指针：一次遍历确定某些特定元素的范围；
2. 反转链表：我喜欢用递归实现；
3. 一些基础的插入删除操作；注意头节点dummy的使用。

<!---more--->
## 82. 删除排序链表中的重复元素 II
https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/

确定重复元素的左右边界；然后维护一个pre变量，因为删除都要从左边界的前一个节点开始删除。

如果left==right，意味没有重复元素，不删除；否则就有重复元素，让pre->next=right->next;

确定左右边界，就是让left=pre->next，然后让right=left，只要right->next不为空且right->next->val == left->val，就让right右移一位。

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
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == NULL){
            return NULL;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* left = head;
        ListNode* right = head;
        ListNode* pre = dummy;

        while(pre->next != NULL){
            left = pre->next;
            right = left;
            while(right->next && right->next->val == left->val){
                right = right->next;
            }

            if(left == right){
                pre = pre->next;
            }else{
                pre->next = right->next;
            }
        }

        return dummy->next;
    }
};
```

## 92. 反转链表 II
https://leetcode-cn.com/problems/reverse-linked-list-ii/

c++，经典快慢指针确定左右范围；然后递归反转链表。注意递归反转链表并不是要反转到末尾，而是n位置处。

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
    ListNode* reverseBetween(ListNode* head, int m, int n) {
        if(head == NULL){
            return NULL;
        }
        if(m == n){
            return head;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;

        ListNode* slow = dummy;
        ListNode* fast = dummy;
        ListNode* slowPrev = dummy;

        //拿到m位置的前驱结点
        for(int i=1;i<m;i++){
            slowPrev = slowPrev->next;
            fast = fast->next;
        }
        //拿到m位置的结点
        slow = slowPrev->next;
        
        //拿到n位置的结点
        for(int i=0;i<=n-m;i++){
            fast = fast->next;
        }
        //拿到n位置后的结点（n+1位置），用于确定reverse的范围以及连接反转后的链表。
        ListNode* end = fast->next;
        //反转链表，拿到反转后的头节点，也就是n位置的结点；
        ListNode* last = reverse(slow,end);

        //前驱结点的后继是反转后的头结点
        slowPrev->next = last;
        //反转后的尾节点其实是m位置的结点，让他的后继指向n+1位置的结点就可以了。
        slow->next = end;
        
        return dummy->next;


    }

    ListNode* reverse(ListNode* head, ListNode* end){
        if(head->next == NULL || head == NULL || head->next == end){
            return head;
        }

        ListNode* last = reverse(head->next,end);
        head->next->next = head;
        head->next = NULL;
        return last;
    }
};
```


## 328. 奇偶链表
https://leetcode-cn.com/problems/odd-even-linked-list/

原地修改，注意o = o->next;
            e->next = o->next;这一步。

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }
        
        ListNode* o = head; // point 
        ListNode* e = head->next; // point 

        ListNode* even = head->next;

        while(o->next && e->next){
            o->next = e->next;
            o = o->next;
            e->next = o->next;
            e = e->next;
        }

        o->next = even;
        return head;

    }
};
```