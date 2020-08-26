---
title: double-pointer-5/6
date: 2020-08-24 23:02:32
tags: [leetcode,双指针]
categories: 
- leetcode
- 双指针
---
LeetCode双指针，第5第6第7页题目。
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

# 881. 救生艇
https://leetcode-cn.com/problems/boats-to-save-people/

先接最重的人，然后如果接了重的人还可以接最轻的人，就两个一起接走；否则只接重的。

所以就是先排序，然后双指针，一个指向轻的，一个指向重的。如果只能接重的，就让right--；如果两个都可以接，就right--和left++。

```C++
class Solution {
public:
    int numRescueBoats(vector<int>& people, int limit) {
        sort(people.begin(),people.end());
        int left=0,right=people.size()-1;
        int ans = 0;

        while(left<=right){
            int r = people[right];
            int l = people[left];
            if(r == limit){
                ans++;
                right--;
            }else if(r < limit){
                if(l+r <= limit){
                    ans++;
                    left++;
                    right--;
                }else{
                    ans++;
                    right--;
                }
            }
        }

        return ans;

    }
};
```

# 142. 环形链表 II
https://leetcode-cn.com/problems/linked-list-cycle-ii/

双指针经典应用。应用labuladong大佬的思路：

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200825213756.png)
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200825213832.png)

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
        if(head == NULL){
            return NULL;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        if(head->next == NULL){
            return NULL;
        }

        while(fast != NULL && fast->next != NULL){
            fast = fast->next->next;
            slow = slow->next;
            //有环
            if(slow == fast){
                slow = head;
                while(slow != fast){
                    fast = fast->next;
                    slow = slow->next;
                }
                return slow;
            }
        }
        //无环
        return NULL;
    }   
};
```

# 28. 实现 strStr()
https://leetcode-cn.com/problems/implement-strstr/

双指针指向两个字符串。如果两个指针指向的字符相等，开始比较，双指针后移；否则指向要比较字符串的指针归零，原字符串指针回到已经进行了比较的字符串长度之前的位置。

最后判断一下j是不是走到要比较的字符串的末尾，如果是，就证明有相等的字符串；否则没有。

```C++
class Solution {
public:
    int strStr(string haystack, string needle) {
        if(needle == ""){
            return 0;
        }
        int i=0,j=0;

        while(i<haystack.size() && j<needle.size()){
            if(haystack[i] == needle[j]){
                i++;
                j++;
            }else{
                i = i-j+1;
                j = 0;
            }
        }
        return j==needle.size()?i-needle.size():-1;

    }
};
```


# 925. 长按键入
https://leetcode-cn.com/problems/long-pressed-name/

同样的，双指针指向两个字符串。如果两个指针指向的字符相等，往前移动；如果不等，判断typed字符串的前一个字符和现在这个字符是不是相等，如果是，继续让指向typed的指针前移；否则证明这时候两个指针指向的字符不等了并且未到末尾，所以返回false。

如果name字符串已经处理完，但typed字符串还没处理完，继续处理；如果typed后面的字符和当前字符不等，就说明不是重复按的输入，直接返回false。

最后是通过指向name指针的位置是不是到name字符串的末尾来判断答案。

```C++
class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        int i=0,j=0;

        if(name.size() > typed.size()){
            return false;
        }

        while(i<name.size() && j<typed.size()){
            if(name[i] == typed[j]){
                i++;j++;
            }
            else if(j>0 && typed[j] == typed[j-1]){
                j++;
            }else{
                return false;
            }
        }

        while(j<typed.size()){
            if(j>0 && typed[j] != typed[j-1]){
                return false;
            }
            j++;
        }

        return i==name.size();
    }
};

```
# 面试题 10.01. 合并排序的数组
https://leetcode-cn.com/problems/sorted-merge-lcci/

归并排序，从大到小放入到原数组中。最后如果B中还有元素没处理完，继续处理。

```C++
class Solution {
public:
    void merge(vector<int>& A, int m, vector<int>& B, int n) {
        

       
        int i=m-1,j=n-1;
        int pos = m+n-1;

        while(i>=0 && j>=0){
            if(A[i] >= B[j]){
                A[pos--] = A[i--];
            }

            else{
                A[pos--] = B[j--];
            }
        }

        // while(i>=0){
        //     A[pos--] = A[i--];
        // }
        while(j>=0){
            A[pos--] = B[j--];
        }

    }
};
```

# 11. 盛最多水的容器
https://leetcode-cn.com/problems/container-with-most-water/

双指针，一个指向左边，一个指向右边；
盛水容积=底*高（左右的最小值）；

如果左高<右高，就让左指针往右移，因为此时如果把右高往左移，面积会变小。否则让右指针往左移。这样逐渐找到最大的容积。

```C++
class Solution {
public:
    int maxArea(vector<int>& height) {
        if(height.size() <= 1){
            return -1;
        }

        int l=0,r=height.size()-1;
        int ans = 0;

        while(l<r){
            int h = min(height[l],height[r]);
            ans = max(ans,(r-l) * h);
            if(height[l] <height[r]){
                l++;
            }else{
                r--;
            }
        }
        return ans;
    }
};
```