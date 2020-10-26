---
title: double-pointer-1/2
date: 2020-08-09 01:50:27
tags: [leetcode,双指针]
categories: 
- leetcode
- 双指针
---

双指针主要分两种，一种是快慢指针，一般用来解决链表问题，比如判断链表是否有环；另一种是左右指针，解决数组或字符串的问题。（来自labuladong的算法小抄）。
<!---more--->

## 763. 划分字母区间
https://leetcode-cn.com/problems/partition-labels/

需要一个map记录遇到的字符的最右端index。

双指针，一个指向开头，一个指向末尾。末尾是上一次右边界和这一次字符的最右idnex. 意思是，如果碰到要计算的字符出现在更右边，就扩大字母区间右边界；否则就以当前字母区间的右边界进行划分。因为这样划分的片段才会尽可能多。

```C++
class Solution {
public:
    vector<int> partitionLabels(string S) {
        vector<int> ans;
        unordered_map<char,int> map;//<strings,index>
        int l=0,r=0;
        for(int i=0;i<S.size();i++){
            map[S[i]] = i;
        }
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

## 15. 三数之和
https://leetcode-cn.com/problems/3sum/

难点在要跳过重复的数字；所以要对数组进行排序。遍历的时候要注意判断跳过重复的数字。外面一层循环，里面一层双指针循环。都要判断跳过重复数字。

```C++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ans;
        int current = 0;
        sort(nums.begin(),nums.end());
        for(int i=0;i<nums.size();i++){
            current = nums[i];
            int l=i+1,r=nums.size()-1;

            //skip the duplicated item
            if(i>0 && nums[i] == nums[i-1]){
                continue;
            }
            //array is sorted, so next is bigger than current, if current>0, a+b+c<0 is impossible
            if(current > 0){
                break;
            }

            while(l<r){
                if(nums[l] + nums[r] + current > 0){
                    r--;
                }else if(nums[l] + nums[r] + current < 0){
                    ++l;
                }else{
                    ans.emplace_back(vector<int>{current,nums[l],nums[r]});
                    l++;r--;
                    //skip the duplicated item
                    while(l<r && nums[l] == nums[l-1]){
                        l++;
                    }
                    //skip the duplicated item
                    while(l<r && nums[r] == nums[r+1]){
                        r--;
                    }
                }
            }
        }
        return ans;

    }
};
```

## 16. 最接近的三数之和
https://leetcode-cn.com/problems/3sum-closest/

和上面类似，先排序；然后再在外层循环里用双指针和二分查找。

```C++
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(),nums.end());
        int ans = 1e7;
        for(int i=0;i<nums.size();i++){
            if(i>0 && nums[i] == nums[i-1]){
                continue;
            }
            int l=i+1,r=nums.size()-1;
            while(l<r){
                int sum = nums[i] + nums[l] + nums[r];
                if(abs(sum-target) < abs(ans - target)){
                    ans = sum;
                }
                if(sum == target){
                    return sum;
                }else if(sum < target){
                    l++;
                }else{
                    r--;
                }
            }
            
        }
        return ans;
    }
};
```

## 26. 删除排序数组中的重复项
双指针经典题目，遇到O(1)空间要求的，基本都是用双指针，一快一慢。
```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.size()<2){
            return nums.size();
        } 
        int i=0,j=1;
        for(int j=1;j<nums.size();j++){
            //慢指针指向要比较的数字，遇到要比较的数字相等的，快指针走一步；如果不相等，要将慢指针往前走一步，然后将不相等的值赋值到慢指针所指处。
            if(nums[i] != nums[j]){
                i++;
                nums[i] = nums[j];
            }
        }
    }
};
```

## 88. 合并两个有序数组
https://leetcode-cn.com/problems/merge-sorted-array/

用多一个数组保存nums1，因为nums1要做修改，会对结果产生影响；

然后对两个数组用双指针，每次取将小的两个数组元素放入nums1，然后将那个数组的指针后移。

最后判断一下两个指针哪个没到尽头，如果还有没放入nums1的，把它放入nums1 。

```C++
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        vector<int> temp(nums1);

        int t1=0,t2=0;
        int i=0;
        while(t1<m && t2<n){
            if(temp[t1] <= nums2[t2]){
                nums1[i++] = temp[t1++];
            }
            else{
                nums1[i++] = nums2[t2++];
            }
        }
        while(t1<m){
            nums1[i++] = temp[t1++];
        }
        while(t2<n){
            nums1[i++] = nums2[t2++];
        }

    }
};
```

## 19. 删除链表的倒数第N个节点
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

这种确定对第N个位置的节点，都可以用双指针。

按照labuladong大佬的教程，可以让快指针先走N步，然后让快慢指针一起走，只要快指针走到末尾，就可以确定第N个位置的节点。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200810191839.png)

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
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummpy = new ListNode(-1);
        dummpy->next = head;
        ListNode* fast;
        ListNode* slow;
        fast = dummpy;
        slow = dummpy;
        while(n>=0){
            fast = fast->next;
            n--;
        }
        while(fast != NULL){
            fast = fast->next;
            slow = slow->next;
        }
        if(slow->next){
            slow->next = slow->next->next;
        }else{
            slow->next = NULL;
        }
        return dummpy->next;
    }
};
```

### 同类题目 剑指 Offer 22. 链表中倒数第k个节点
https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/
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
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode* slow=head;
        ListNode* fast=head;

        while(k-->0){
            fast = fast->next;
        }
        while(fast != NULL){
            fast = fast->next;
            slow = slow->next;
        }
        return slow;

    }
};
```

## 面试题 17.11. 单词距离
https://leetcode-cn.com/problems/find-closest-lcci/

遍历一遍，记录两个单词的index。然后计算index之间的距离，取距离的最小值。

```C++
class Solution {
public:
    int findClosest(vector<string>& words, string word1, string word2) {
        int index1=-1,index2=-1;
        int ans = INT_MAX;
        bool change = false;
        for(int i=0;i<words.size();i++){
            if(words[i] == word1){
                index1 = i;
                change = true;
            }
            if(words[i] == word2){
                index2 = i;
                change = true;
            }
            if(index1 != -1 && index2 != -1 && change){
                ans = min(ans,abs(index2-index1));
                change = false;
            }
        }
        return ans;
    }
};
```

## 923. 三数之和的多种可能
https://leetcode-cn.com/problems/3sum-with-multiplicity/

```C++
class Solution {
public:
    using LL=long long;
    int threeSumMulti(vector<int>& A, int target) {
        LL ans=0;
        int mod = 1e9+7;
        sort(A.begin(),A.end());
        for(int i=0;i<A.size();i++){
            int j=i+1,k=A.size()-1;
            int remain = target - A[i];
            //双指针
            while(j<k){
                //排序二分双指针，左指针右移
                if(A[j] + A[k] < remain){
                    j++;
                }
                //排序二分双指针，右指针左移
                else if(A[j] + A[k] > remain){
                    k--;
                }
                //找到和了， 判断两个元素是否相等。如果不等，找到这两个元素有多少个。
                //组合可能=左元素个数*右元素个数
                else if(A[j] + A[k] == remain && A[j]!=A[k]){
                    int left = 1,right = 1;
                    while(j+1<k && A[j] == A[j+1]){
                        left++;j++;
                    }
                    while(k-1>j && A[k] == A[k-1]){
                        right++;k--;
                    }
                    ans += left*right;
                    ans %= mod;
                    j++;
                    k--;
                }
                //找到和，两个元素都相等，他们的组合数就是(k-j+1) * (k-j) / 2;
                else{
                    ans += (k-j+1) * (k-j) / 2;
                    ans %= mod;
                    break;
                }
            }
        }
        return ans;

    }
};
```

## 234. 回文链表
https://leetcode-cn.com/problems/palindrome-linked-list/

找到链表的中点，从中点开始反转链表。

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
    bool isPalindrome(ListNode* head) {
        if(head == NULL || head->next == NULL){
            return true;
        }
        ListNode* slow = head;
        ListNode* fast = head;

        while(fast->next != NULL && fast->next->next != NULL){
            fast = fast->next->next;
            slow = slow->next;
        }
        slow = reverse(slow->next);

        while(slow!=NULL){
            if(head->val != slow->val){
                return false;
            }
            head = head->next;
            slow = slow->next;
        }
        return true;

    }

    ListNode* reverse(ListNode* head){
        if(head->next == NULL){
            return head;
        }
        ListNode* last = reverse(head->next);
        head->next->next = head;
        head->next = NULL;
        return last;
    }
};
```

## 125. 验证回文串
https://leetcode-cn.com/problems/valid-palindrome/
这道题是数据预处理比较麻烦，要将所有字母和数字都处理成小写，去掉不必要的符号。
然后再用双指针判断回文。

```C++
class Solution {
public:
    bool isPalindrome(string s) {
        string str="";
        for(auto c:s){  
            if(('A'<=c && c<='Z') || ('a'<=c && c<='z') || ('0'<=c && c<='9')){
                str+=tolower(c);
            }
        }
        int i=0,j=str.size()-1;
        while(i<j){
            if(str[i] != str[j]){
                return false;
            }
            i++;j--;
        }
        return true;

    }
};
```

## 986. 区间列表的交集
https://leetcode-cn.com/problems/interval-list-intersections/

双指针，指向要对比的元素。区间套定理，比较左右区间端点的最小值和最大值，判断有没有重合。

```C++
class Solution {
public:
    vector<vector<int>> intervalIntersection(vector<vector<int>>& A, vector<vector<int>>& B) {
        vector<vector<int>> ans;
        int i=0,j=0;

        while(i<A.size() && j<B.size()){
            int start = max(A[i][0],B[j][0]);
            int end = min(A[i][1],B[j][1]);
            if(start <= end){
                ans.push_back({start,end});
            }
            //比较右端点，右端点小的指针前移
            if(A[i][1] < B[j][1]){
                i++;
            }else{
                j++;
            }
            
        }
        return ans;
    }
};
```

## 845. 数组中的最长山脉
https://leetcode-cn.com/problems/longest-mountain-in-array/

通过比较左右元素，找到山脉数组中间最高的值。然后用双指针确定左右起点的范围。山脉数组的长度就是r-l+1 。

```C++
class Solution {
public:
    int longestMountain(vector<int>& A) {
        if(A.size()<2){
            return 0;
        }

        int ans=0;
        int l=-1,r=-1;

        for(int i=1;i<A.size()-1;i++){
            if(A[i-1]<A[i] && A[i+1] < A[i]){
                l = i-1;
                r = i+1;
                while(l>0 && A[l-1] < A[l]){
                    l--;
                }
                while(r < A.size()-1 && A[r]>A[r+1]){
                    r++;
                }
                ans = max(ans,(r-l+1));
            }
        }
        return ans;
        
        
    }
};
```