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
4. 一些需要对当前节点进行调换位置的时候，可以以这个节点的前一个节点的next作为处理结果。（147题）

<!---more--->
## 2. 两数相加
https://leetcode-cn.com/problems/add-two-numbers/

这个没啥说的，就像大数加法一样，只不过都换成了链表。依次维护进位相加就行。

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int carry = 0;
        ListNode* head = new ListNode(-1);
        ListNode* cur = head;
        while(l1 || l2 || carry){
            int sum = carry;
            if(l1){
                sum+=l1->val;
                l1 = l1->next;
            }
            if(l2){
                sum+=l2->val;
                l2 = l2->next;
            }
            carry = sum/10;
            sum = sum%10;
            cur->next = new ListNode(sum);
            cur = cur->next;
        }
        return head->next;
    }
};
```

## 206. 反转链表
https://leetcode-cn.com/problems/reverse-linked-list/

据说是频率最高的题；维护一个prev结点，每次将curr结点的next指针指向prev；

更新prev和curr就好了。注意return是prev

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
    ListNode* reverseList(ListNode* head) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* prev = nullptr;
        ListNode* curr = head;

        while(curr){
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }
};
```

## 21. 合并两个有序链表
https://leetcode-cn.com/problems/merge-two-sorted-lists/

归并排序嘛，哪个小用哪个接在后面，更新接了的那个链表的当前结点。

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
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        if(l1 == nullptr){
            return l2;
        }
        if(l2 == nullptr){
            return l1;
        }

        ListNode* dummy = new ListNode(-1);
        ListNode* curr = dummy;

        while(l1 && l2){
            if(l1->val < l2->val){
                curr->next = l1;
                l1 = l1->next;
            }else{
                curr->next = l2;
                l2 = l2->next;
            }
            curr = curr->next;
        }

        while(l1){
            curr->next = l1;
            l1 = l1->next;
            curr = curr->next;
        }

        while(l2){
            curr->next = l2;
            l2 = l2->next;
            curr = curr->next;
        }

        return dummy->next;
    }
};
```

## 92. 反转链表 II
https://leetcode-cn.com/problems/reverse-linked-list-ii/

确定链表反转的范围，将最左边的前一个定义为prev，最左边的那个作为curr；
然后每次反转都是将curr->next翻转接到prev->next;
从left到right共需要反转right-left次。

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
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* prev = dummy;
        ListNode* curr = head;

        for(int i=1; i<left; i++){
            prev = prev->next;
        }
        curr = prev->next;
        for(int i=left; i<right; i++){
            auto next = curr->next;
            curr->next = next->next;
            next->next = prev->next;
            prev->next = next;
        }
        return dummy->next;
    }
};
```

## 25. K 个一组翻转链表
https://leetcode-cn.com/problems/reverse-nodes-in-k-group/

利用上一题的方法对每个k组里面进行反转（共k-1次）；反转后外层需要更新prev和curr（分成len/k组，每个组反转后更新，所以共len/k次）；
因为反转后尾部是curr；所以外层prev = curr； curr = prev->next;

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
    ListNode* reverseKGroup(ListNode* head, int k) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* curr = head;
        int len = 0;
        while(curr){
            len++;
            curr = curr->next;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* prev = dummy;
        curr = head;

        for(int i=0; i<len/k; i++){
            for(int j=1; j<k; j++){
                auto next = curr->next;
                curr->next = next->next;
                next->next = prev->next;
                prev->next = next;
            }
            prev = curr;
            curr = curr->next;
        }

        return dummy->next;
    }
};
```

## 143. 重排链表
https://leetcode-cn.com/problems/reorder-list/

快慢指针找中点，然后从中点开始reverse链表；最后按照顺序接入。

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
    void reorderList(ListNode* head) {
        if(head == nullptr || head->next == nullptr){
            return ;
        }

        ListNode* slow = head;
        ListNode* fast = head;
        while(fast->next && fast->next->next){
            slow = slow->next;
            fast = fast->next->next;
        }
        auto slowNext = slow->next;
        slow->next = nullptr;
        ListNode* reverseHead = reverseList(slowNext);

        ListNode* l1 = head;
        ListNode* l2 = reverseHead;

        while(l1 && l2){
            auto l1next = l1->next;
            auto l2next = l2->next;

            l1->next = l2;
            l2->next = l1next;

            l1 = l1next;
            l2 = l2next; 
        }

    }

    ListNode* reverseList(ListNode* head){
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* prev = nullptr;
        ListNode* curr = head;

        while(curr){
            auto next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```

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


## 147. 对链表进行插入排序
https://leetcode-cn.com/problems/insertion-sort-list/

将要排序的节点（比前一个节点小）拉到前面开始比较；如果比前面的节点大，就继续往后面比较；否则就将他插入到当前比较的节点的后面。

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
    ListNode* insertionSortList(ListNode* head) {
        if(head == NULL || head->next == NULL){
            return head;
        }

        ListNode*  dummy = new ListNode(INT_MIN);
        dummy->next = head;
        ListNode* prev = NULL;

        //真正要处理的的节点是curP->next, curP means current-previous node
        ListNode* curP = head;
        while(curP != NULL && curP->next != NULL){
            prev = dummy;
            //前面的已经有序了，如果下一个比现在的要大，就不用排序了，往后移动指针
            if(curP->next->val >= curP->val){
                curP = curP->next;
                continue;
            }else{
                while(curP->next->val > prev->next->val){
                    prev = prev->next;
                }
                auto curNext = curP->next;
                curP->next = curNext->next;
                curNext->next = prev->next;
                prev->next = curNext;
            }
           
        }
        return dummy->next;
    }
};
```

## 61. 旋转链表
https://leetcode-cn.com/problems/rotate-list/

旋转链表其实就是将后面k个元素挪到前面；先用长度对k取余数，看要移动哪几个元素；
然后通过快慢指针定位到后k个元素，将后k个元素移动到前面，原头部接到后面。

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

        int len = 0;
        ListNode* cur = head;
        while(cur){
            cur = cur->next;
            len++;
        }
        
        k = k%len;
        //k == 0 不需要移动
        if(k == 0){
            return head;
        }
        
        ListNode* slow = head;
        ListNode* fast = head;
        while(k-- && fast){
            fast = fast->next;
        }

        while(fast && fast->next){
            fast = fast->next;
            slow = slow->next;
        }

        auto next = slow->next;
        slow->next = NULL;
        fast->next = head;
        return next;
    }
};
```

## 86. 分隔链表
https://leetcode-cn.com/problems/partition-list/

将大于x和小于等于x的分成两个链表，分别将原节点接到他们对应的链表上。然后再将这两个链表合并。

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
    ListNode* partition(ListNode* head, int x) {
        ListNode* bigHead = new ListNode(-1);
        ListNode* smallHead = new ListNode(-1);

        ListNode* curr = head;
        ListNode* currBig = bigHead;
        ListNode* currSmall = smallHead;
        while(curr){
            if(curr->val < x){
                currSmall->next = curr;
                currSmall = currSmall->next;
            }else{
                currBig->next = curr;
                currBig = currBig->next;
            }
            curr = curr->next;
        }
        currSmall->next = bigHead->next;
        currBig->next = nullptr;
        return smallHead->next;

    }
};
```

## 147. 对链表进行插入排序
https://leetcode-cn.com/problems/insertion-sort-list/

用一个结点记住排好序的最后一个节点，然后每次比较的时候都从这个节点开始比较，如果当前节点是小于排好序的节点的，就要将它插入到前面。

每次插入排序，都从头开始找到比当前大的第一个节点，然后将它插入到它的前面。

插入完成后，因为当前节点已经挪走了，所以要再更新lastSorted的next为current->next；

最后挪动一下curr节点到lastSorted的next。

```C++
class Solution {
public:
    ListNode* insertionSortList(ListNode* head) {
        if (head == nullptr) {
            return head;
        }
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* lastSorted = head;
        ListNode* curr = head->next;
        while (curr != nullptr) {
            if (lastSorted->val <= curr->val) {
                lastSorted = lastSorted->next;
            } else { 
                ListNode *prev = dummyHead;
                while (prev->next->val <= curr->val) {
                    prev = prev->next;
                }
                lastSorted->next = curr->next;
                curr->next = prev->next;
                prev->next = curr;
            }
            curr = lastSorted->next;
        }
        return dummyHead->next;
    }
};

```
