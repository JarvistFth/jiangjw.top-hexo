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

## 146. LRU缓存机制
https://leetcode-cn.com/problems/lru-cache/

哈希表+双向链表；每次都把用过的移到链表头部，位置不够时删除链表尾部；哈希表根据key可以O(1)找到对应的链表节点。

```C++
class Node{
public:
    int key;
    int val;
    Node* prev;
    Node* next;

    Node() = default;
    Node(int key, int val):key(key),val(val),prev(nullptr),next(nullptr){

    }
    ~Node() = default;
};

class LRUCache {
public:
    int capacity;
    Node* head;
    Node* tail;
    unordered_map<int,Node*> cache;
    LRUCache(int capacity):capacity(capacity) {
        head = new Node(-1,-1);
        tail = new Node(-1,-1);
        head->next  = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if(cache.count(key)){
            put(key,cache[key]->val);
            return cache[key]->val;
        }
        return -1;
    }
    
    void put(int key, int value) {
        Node* newNode = new Node(key,value);
        //如果已经有这个key了，移到表头
        if(cache.count(key)){
            //delete one
            auto n = deleteOne(cache[key]);
            delete n;
            //add to first
            addToFirst(newNode);
            //update cache
            cache[key] = newNode;
        }else{
            //没有key，要加，这时候满了，要先删掉最后的
            if(cache.size() == capacity){
                //delete tail

                auto r = deleteTail();

                //remove cache
                cache.erase(r->key);
                delete r;

                //add to first
                addToFirst(newNode);

                //update cache
                cache[key] = newNode;
            }else{
                //还没满，直接加就行

                //add to first
                addToFirst(newNode);

                //update cache
                cache[key] = newNode;

            }
        }
    }


    Node* deleteOne(Node* n){
        n->prev->next = n->next;
        n->next->prev = n->prev;
        return n;
    }

    Node* deleteTail(){
        Node* n = tail->prev;
        auto prev = deleteOne(n);
        return prev;
    }

    void addToFirst(Node* n){
        auto hnext = head->next;
        head->next = n;
        n->next = hnext;
        n->prev = head;
        hnext->prev = n;
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

##  143. 重排链表
https://leetcode-cn.com/problems/reorder-list/

快慢指针找到中点的靠后一位，从这一位开始反转链表；

然后将两个反转后的链表依次拼接

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
        if(head == nullptr){
            return ;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast->next && fast->next->next){
            slow = slow->next;
            fast = fast->next->next;
        }

        auto slowNext = slow->next;
        slow->next = nullptr;
        auto reverseHead = reverse(slowNext);

        ListNode* p = head;
        ListNode* q = reverseHead;

        while(p && q){
            auto pnext = p->next;
            auto qnext = q->next;

            p->next = q;
            q->next = pnext;

            p = pnext;
            q = qnext;
        }

    }


    ListNode* reverse(ListNode* head){
        if(head == nullptr){
            return nullptr;
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

## 23. 合并K个升序链表
https://leetcode-cn.com/problems/merge-k-sorted-lists/

首先通过分治法，将多个链表拆分成两个一组；然后对这两个链表进行普通的归并排序。

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

    ListNode* splitLists(vector<ListNode*>& lists, int l, int r){
        if(lists.empty()){
            return nullptr;
        }
        if(l >= r){
            return lists[l]; 
        }
        int mid = l+(r-l)/2;
        auto left = splitLists(lists, l, mid);
        auto right = splitLists(lists, mid+1, r);
        return mergeTwoLists(left, right);
    }

    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2){
        ListNode* dummy = new ListNode(-1);
        ListNode* curr = dummy;
        while(l1 && l2){
            if(l1->val < l2->val){
                curr->next = new ListNode(l1->val);
                l1 = l1->next;
            }else{
                curr->next = new ListNode(l2->val);
                l2 = l2->next;
            }
            curr = curr->next;
        }

        while(l1){
            curr->next = new ListNode(l1->val);
            l1 = l1->next;
            curr = curr->next;
        }

        while(l2){
            curr->next = new ListNode(l2->val);
            l2 = l2->next;
            curr = curr->next;
        }
        return dummy->next;
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        
        return splitLists(lists, 0, lists.size()-1);
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
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == nullptr){
            return nullptr;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* left = head;
        ListNode* right = head;
        ListNode* prev = dummy;

        while(prev->next){
            left = prev->next;
            right = left;
            while(right->next && right->next->val == left->val){
                right = right->next;
            }

            if(left != right){
                while(left != right){
                    auto next = left->next;
                    delete left;
                    left = next;
                }
                prev->next = right->next;
            }else{
                prev = prev->next;
            }
        }
        return dummy->next;
    }
};
```

## 234. 回文链表
https://leetcode-cn.com/problems/palindrome-linked-list/

快慢指针找中点，从中点后一点开始反转链表，然后判断反转后的链表和中点前的链表的值是否相等。

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
    bool isPalindrome(ListNode* head) {
        if(head == nullptr){
            return false;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast && fast->next){
            fast = fast->next->next;
            slow = slow->next;
        }

        auto reverseHead = reverse(slow);

        ListNode* p = head;
        ListNode* q = reverseHead;
        while(p && q){
            if(p->val != q->val){
                return false;
            }else{
                p = p->next;
                q = q->next;
            }
        }
        return true;
    }

    ListNode* reverse(ListNode* head){
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* curr = head;
        ListNode* prev = nullptr;
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

## 剑指 Offer 06. 从尾到头打印链表
https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/

用栈、reverse、reverseList都可以做。这里比较高效的方法就是通过给数组对应的下表赋值。

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
    vector<int> reversePrint(ListNode* head) {
        ListNode* p = head;
        int n = 0;
        while(p){
            n++;
            p = p->next;
        }
        vector<int> ans(n);
        p = head;
        while(p){
            ans[--n] = p->val;
            p = p->next; 
        }
        return ans;
    }
};
```

## 148. 排序链表
https://leetcode-cn.com/problems/sort-list/

bottom-up写法可以租到O(nlogn)时间复杂度和O(1)空间复杂度；这里用了分治，引入了栈的空间，所以不是O(1)事件复杂度。

快慢指针找中点，从中点开始分治两边，对两边的链表从上到下做merge。

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

    ListNode* splitLists(ListNode* head){
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast->next && fast->next->next){
            fast = fast->next->next;
            slow = slow->next;
        }

        auto slownext = slow->next;
        slow->next = nullptr;

        auto l = splitLists(head);
        auto r = splitLists(slownext);

        return mergeTwoLists(l, r);
    }

    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2){
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
        auto ret = dummy->next;
        delete dummy;
        return ret;
    }
    ListNode* sortList(ListNode* head) {
        return splitLists(head);
    }
};
```

## 19. 删除链表的倒数第 N 个结点
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

快慢指针，快指针先走n步；然后快慢指针一起走；快指针走到末尾时，将慢指针后一个结点删除。

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
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if(head == nullptr ){
            return head;
        }
        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* fast = dummy;
        while(n-- && fast){
            fast = fast->next;
        }

        ListNode* slow = dummy;

        while(fast && fast->next){
            slow = slow->next;
            fast = fast->next;
        }

        auto next = slow->next;
        if(next){
            slow->next = next->next;
        }
        delete next;
        auto ret = dummy->next;
        delete dummy;
        return ret;
    }
};
```


## 141. 环形链表
https://leetcode-cn.com/problems/linked-list-cycle/

快慢指针，快指针走2步，慢指针走1步；有环必定会碰上；没有环会退出循环。

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
        if(head == nullptr){
            return false;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast && fast->next){
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

## 83. 删除排序链表中的重复元素
https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/

排序链表，每次删除前只要比较一下当前元素和前一个元素值是否一致即可，一致则删除。

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
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* prev = head;
        ListNode* curr = head->next;

        while(curr){
            if(curr->val == prev->val){
                auto next = curr->next;
                prev->next = next;
                delete curr;
                curr = next;
            }else{
                prev = curr;
                curr = curr->next;
            }
        }
        return head;

    }
};
```

## 142. 环形链表 II
https://leetcode-cn.com/problems/linked-list-cycle-ii/

快慢指针，快指针走2步，慢指针走1步；有环必定会碰上；没有环会退出循环。

碰上时，让慢指针指向表头，从头开始一起走，再相遇时即为环的起点。

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
    ListNode *detectCycle(ListNode *head) {
        if(head == nullptr){
            return head;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(fast && fast->next){
            fast = fast->next->next;
            slow = slow->next;

            if(fast == slow){
                slow = head;
                while(fast != slow){
                    fast = fast->next;
                    slow = slow->next;
                }
                return slow;
            }
        }
        return nullptr;
    }
};
```

## 138. 复制带随机指针的链表
https://leetcode-cn.com/problems/copy-list-with-random-pointer/

三步操作：
1. 当前结点就地复制在本结点的next；
2. 调整复制后的random指针：将复制后的random指针指向原random指针的next；
3. 断开链表：原链表和复制链表分开。curr->next指向curr->next->next；cpCurr->next指向cpCurr->next->next；（为了避免复制后的结点没有next，即为复制后的表尾，所以要判空）。

```C++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(head == nullptr){
            return nullptr;
        }
        
        Node* curr = head;
        while(curr){
            auto next = curr->next;
            curr->next = new Node(curr->val);
            curr->next->next = next;
            curr = next;
        }
        curr = head;
        while(curr){
            if(curr->random){
                curr->next->random = curr->random->next;
            }
            curr = curr->next->next;
        }

        curr = head;
        Node* cpCurr = curr->next;
        Node* cpHead = head->next;


        while(curr){
            curr->next = curr->next->next;
            curr = curr->next;
            if(cpCurr->next){
                cpCurr->next = cpCurr->next->next;
                cpCurr = cpCurr->next;
            }
        }

        return cpHead;
    }
};
```

## 445. 两数相加 II
https://leetcode-cn.com/problems/add-two-numbers-ii/

和两数相加I很像，只是这次要从链表尾开始加。两个办法：
1. 用栈；从表头开始入栈到表尾；然后从栈顶开始取数相加；
2. 反转链表：反转后按照1的方式去加，然后再反转。

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
        stack<int> s1,s2;
        ListNode* dummy = new ListNode(-1);
        ListNode* curr = dummy;
        while(l1){
            s1.push(l1->val);
            l1 = l1->next;
        }

        while(l2){
            s2.push(l2->val);
            l2 = l2->next;
        }
        int carry = 0;
        while(!s1.empty() || !s2.empty() || carry){
            int sum = carry;
            if(!s1.empty()){
                sum += s1.top();
                s1.pop();
            }

            if(!s2.empty()){
                sum += s2.top();
                s2.pop();
            }
            carry = sum/10;
            auto next = dummy->next;
            dummy->next = new ListNode(sum%10);
            dummy->next->next = next;
        }
        return dummy->next;
    }
};
```

## 面试题 02.03. 删除中间节点
https://leetcode-cn.com/problems/delete-middle-node-lcci/

让当前结点的值为next结点的值，然后再把当前的node->next连到next->next，最后把next删除，

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
    void deleteNode(ListNode* node) {
        if(node == NULL){
            return ;
        }

        auto next = node->next;
        node->val = next->val;
        node->next = next->next;
        delete next;
    }
};
```

## 328. 奇偶链表
https://leetcode-cn.com/problems/odd-even-linked-list/

给奇数结点建一个指针odd，偶数结点建一个指针even。每次接的时候将odd->next = even->next；然后更新odd指针为下一个odd；even同理。

注意当odd或even在末尾最后一个结点的时候，就不需要再拼接了。这时候只需将odd->next和even链表的head接起来。

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
        ListNode* odd = head;
        ListNode* even = head->next;
        ListNode* evenHead = even;

        while(odd->next && even->next){
            odd->next = even->next;
            odd = odd->next;

            even->next = odd->next;
            even = even->next;
        }

        odd->next = evenHead;
        return head;
    }
};
```

## 24. 两两交换链表中的节点
https://leetcode-cn.com/problems/swap-nodes-in-pairs/

正常处理拼接关系。注意后面的back指针指向的prev->next时可能为空。

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
    ListNode* swapPairs(ListNode* head) {
        if(head == nullptr || head->next == nullptr){
            return head;
        }

        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* prev = dummy;
        ListNode* front = head;
        ListNode* back = head->next;

        while(front && back){
            auto next = back->next;
            prev->next = back;
            back->next = front;
            front->next = next;

            prev = front;
            front = prev->next;
            back = prev->next?prev->next->next:nullptr;
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

## 剑指 Offer 36. 二叉搜索树与双向链表
https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/

二叉搜索树，所以肯定是中序遍历。先定义一个头指针和尾指针。

中序遍历的作用，是将树的结点连接成双向链表；所以中序遍历左子树后，左子树已经连好成为了双向链表；接下来对根节点，要做的就是将根节点接到双向链表后。

所以将尾指针的right连接到根节点上；然后更新尾指针，再把根节点的left连到原来的尾指针指向的结点上。

需要注意如果原来head为空，就先创建head结点，同时更新尾指针指向head。

```C++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* left;
    Node* right;

    Node() {}

    Node(int _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }

    Node(int _val, Node* _left, Node* _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
public:
    Node* head = NULL;
    Node* tail = NULL;
    Node* treeToDoublyList(Node* root) {
        if(root == NULL){
            return NULL;
        }

        dfs(root);
        head->left = tail;
        tail->right = head;
        return head;
    }

    void dfs(Node* root){
        if(root == NULL){
            return ;
        }

        dfs(root->left);

        if(head == NULL){
            head = root;
            tail = head;
        }else{
            tail->right = root;
            auto prev = tail;
            tail = tail->right;
            tail->left = prev;
        }

        dfs(root->right);


    }
};
```

## 160. 相交链表
https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

定义两个指针，当走到末尾nullptr的时候，指向另一个的链表头。只要有环，他们一定会相遇，并且相遇的地方就是起点。（走的路程一样）

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
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* pA = headA;
        ListNode* pB = headB;

        while(pA != pB){
            pA = pA == NULL?headB:pA->next;
            pB = pB == NULL?headA:pB->next;
        }
        return pA;
    }
};
```

## 109. 有序链表转换二叉搜索树
https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/

快慢指针找中点，取出中点后，将中点val值作为rootval；然后递归左右链表作为左右子树。

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
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        if(head == nullptr){
            return nullptr;
        }

        ListNode* fast = head;
        ListNode* slow = head;
        ListNode* prev = nullptr;

        while(fast && fast->next){
            fast = fast->next->next;
            prev = slow;
            slow = slow->next;
        }
        TreeNode* root = new TreeNode(slow->val);
        if(prev){
            prev->next = nullptr;
            root->left = sortedListToBST(head);
            root->right = sortedListToBST(slow->next);
        }
        return root;
        
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
