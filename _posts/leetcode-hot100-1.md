---
title: leetcode-hot100-1
date: 2020-09-14 20:02:24
categories:
- leetcode
- hot100
tags: [leetcode ,hot100]
keywords: [leetcode,hot100]
---

# 1. 两数之和
https://leetcode-cn.com/problems/two-sum/

用一个map记录数组中元素对应的下标，每次遍历新的元素的时候到map里面去找有没有加起来等于target的元素，如果有，将map记录的下标和当前下标放入ans中。

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> map;
        vector<int> ans;
        for(int i=0;i<nums.size();i++){
            int remain = target - nums[i];
            if(map.count(remain)){
                ans.push_back(i);
                ans.push_back(map[remain]);
            }
            map[nums[i]] = i;
        }
        return ans;
    }
};
```

# 2. 两数相加

维护一个进位变量，当链表有值的时候或者进位不为0的时候，循环进行加法。

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int t = 0;
        ListNode* dummy = new ListNode(-1);
        ListNode* current = dummy;

        while(l1 || l2 || t){
            int sum = t;
            if(l1){
                sum += l1->val;
                l1 = l1->next;
            }
            if(l2){
                sum += l2->val;
                l2 = l2->next;
            }
            t = sum/10;
            sum = sum%10;
            current->next = new ListNode(sum);
            current = current->next;
        }
        return dummy->next;
    }
};
```

# 3. 无重复字符的最长子串
https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

滑动窗口，遇到重复字符时开始出窗，出窗后更新最长子串的长度。

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
                int ans = 0;
        unordered_map<char,int> window;

        int left=0,right=0;

        while(right<s.size()){
            char c = s[right];
            window[c]++;
            right++;
            
            while(window[c]>1){
                char d = s[left];
                window[d]--;
                left++;
            }
            ans = max(ans,right-left);
        }
        return ans;
    }
};
```

# 4. 寻找两个正序数组的中位数
https://leetcode-cn.com/problems/median-of-two-sorted-arrays/

一开始想到归并排序，然后复杂度O(M+N)，血崩。

```C++
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int len1 = nums1.size(), len2 = nums2.size();
        int mid = len1+len2;
        vector<int> vec((len1+len2),0);
        int i=0,j=0;
        
        int index = 0;
        double ans = 0;

        while(i<len1 && j<len2){
            int n1 = nums1[i], n2 = nums2[j];
            if(n1 < n2){
                i++;
                vec[index++] = n1;
            }else{
                j++;
                vec[index++] = n2;
            }
        }

        while(i<len1){
            vec[index++] = nums1[i++];
        }

        while(j<len2){
            vec[index++] = nums2[j++];
        }

        ans = mid%2==1?vec[mid/2]:(vec[mid/2] + vec[mid/2-1])/2.0;
        return ans;
    }
};
```

题目要求O(log(m+n));

真大佬解法：

* 这道题让我们求两个有序数组的中位数，而且限制了时间复杂度为O(log (m+n))，看到这个时间复杂度，自然而然的想到了应该使用二分查找法来求解。那么回顾一下中位数的定义，如果某个有序数组长度是奇数，那么其中位数就是最中间那个，如果是偶数，那么就是最中间两个数字的平均值。这里对于两个有序数组也是一样的，假设两个有序数组的长度分别为m和n，由于两个数组长度之和 m+n 的奇偶不确定，因此需要分情况来讨论，对于奇数的情况，直接找到最中间的数即可，偶数的话需要求最中间两个数的平均值。为了简化代码，不分情况讨论，我们使用一个小trick，我们分别找第 (m+n+1) / 2 个，和 (m+n+2) / 2 个，然后求其平均值即可，这对奇偶数均适用。加入 m+n 为奇数的话，那么其实 (m+n+1) / 2 和 (m+n+2) / 2 的值相等，相当于两个相同的数字相加再除以2，还是其本身。

* 这里我们需要定义一个函数来在两个有序数组中找到第K个元素，下面重点来看如何实现找到第K个元素。首先，为了避免产生新的数组从而增加时间复杂度，我们使用两个变量i和j分别来标记数组nums1和nums2的起始位置。然后来处理一些边界问题，比如当某一个数组的起始位置大于等于其数组长度时，说明其所有数字均已经被淘汰了，相当于一个空数组了，那么实际上就变成了在另一个数组中找数字，直接就可以找出来了。还有就是如果K=1的话，那么我们只要比较nums1和nums2的起始位置i和j上的数字就可以了。难点就在于一般的情况怎么处理？因为我们需要在两个有序数组中找到第K个元素，为了加快搜索的速度，我们要使用二分法，对K二分，意思是我们需要分别在nums1和nums2中查找第K/2个元素，注意这里由于两个数组的长度不定，所以有可能某个数组没有第K/2个数字，所以我们需要先检查一下，数组中到底存不存在第K/2个数字，如果存在就取出来，否则就赋值上一个整型最大值。如果某个数组没有第K/2个数字，那么我们就淘汰另一个数字的前K/2个数字即可。有没有可能两个数组都不存在第K/2个数字呢，这道题里是不可能的，因为我们的K不是任意给的，而是给的m+n的中间值，所以必定至少会有一个数组是存在第K/2个数字的。最后就是二分法的核心啦，比较这两个数组的第K/2小的数字midVal1和midVal2的大小，如果第一个数组的第K/2个数字小的话，那么说明我们要找的数字肯定不在nums1中的前K/2个数字，所以我们可以将其淘汰，将nums1的起始位置向后移动K/2个，并且此时的K也自减去K/2，调用递归。反之，我们淘汰nums2中的前K/2个数字，并将nums2的起始位置向后移动K/2个，并且此时的K也自减去K/2，调用递归即可。

```C++
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size(), n = nums2.size();
        int left = (m+n+1)/2;
        int right = (m+n+2)/2;
        double ans = (findKth(nums1,nums2,0,0,left) + findKth(nums1,nums2,0,0,right))/2.0;
        return ans;
    }

    int findKth(vector<int>& nums1, vector<int>& nums2, int i, int j, int k){
        if(i>=nums1.size()){
            return nums2[j+k-1];
        }
        if(j>=nums2.size()){
            return nums1[i+k-1];
        }
        if(k == 1){
            return min(nums1[i],nums2[j]);
        }

        int midval1 = (i+k/2-1) < nums1.size()? nums1[i+k/2-1]:INT_MAX;
        int midval2 = (j+k/2-1) < nums2.size()? nums2[j+k/2-1]:INT_MAX;

        return midval1<midval2?findKth(nums1,nums2,i+k/2,j,k-k/2):findKth(nums1,nums2,i,j+k/2,k-k/2);
    }
};
```

# 5. 最长回文子串
https://leetcode-cn.com/problems/longest-palindromic-substring/

对字符串的每一个字符，都枚举作为回文串的中心字符，然后进行两边扩散。

对于左右相邻相等的字符，他们可能共同作为回文子串的中心，因此需要将l和r分别设为他们两个；这里通过判断他们是否相等，如果相等的话，让r++；

对于左右不相等的，他们就只可能以当前字符作为中心，让l和r都等于i。

一旦发现左右字符不相等，立刻退出循环，然后维护最长回文子串的长度，记录当前的起始位置和结束位置，通过substr截取就可以返回答案了。

```C++
class Solution {
public:
    string longestPalindrome(string s) {
        if(s.empty()){
            return "";
        }
        int begin=0,len=0;

        for(int i=0;i<s.size();i++){
            int l=i,r=i;

            while(r<s.size()-1 && s[r] == s[r+1]){
                r++;
            }

            while(l>=0&&r<s.size()){
                if(s[l] == s[r]){
                    r++;l--;
                }else{
                    break;
                }
            }
            r--;l++;
            if((r-l+1) > len){
                begin = l;
                len = r-l+1;
            }
            
        }
        return s.substr(begin,len);   
    }  
};
```

# 10. 正则表达式匹配
https://leetcode-cn.com/problems/regular-expression-matching/

感觉正解还是用dp比较好。。

```C++
class Solution {
public:
    bool isMatch(string s, string p) {
        return isMatch(s.c_str(), p.c_str());
    }
    
    bool isMatch(const char* s, const char* p) {
        if(*p == 0) return *s == 0;
        
        auto first_match = *s && (*s == *p || *p == '.');
        
        if(*(p+1) == '*'){
            return isMatch(s, p+2) //*匹配0个
            || 
            //匹配1个
            (first_match && isMatch(++s, p));
        }
        else{
            return first_match && isMatch(++s, ++p);
        }
    }
};
```
dp解法：
```C++
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.size();
        int n = p.size();

        auto matches = [&](int i, int j) {
            if (i == 0) {
                return false;
            }
            if (p[j - 1] == '.') {
                return true;
            }
            return s[i - 1] == p[j - 1];
        };

        vector<vector<int>> f(m + 1, vector<int>(n + 1));
        f[0][0] = true;
        for (int i = 0; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (p[j - 1] == '*') {
                    f[i][j] |= f[i][j - 2];
                    if (matches(i, j - 1)) {
                        f[i][j] |= f[i - 1][j];
                    }
                }
                else {
                    if (matches(i, j)) {
                        f[i][j] |= f[i - 1][j - 1];
                    }
                }
            }
        }
        return f[m][n];
    }
};
```

# 11. 盛最多水的容器
https://leetcode-cn.com/problems/container-with-most-water/

双指针，指向最低高度的一侧，此时面积等于左右指针围成的底乘最低高度；然后判断一下是否大于最大面积。是就更新最大面积；

然后双指针从矮的那一侧开始移动。

```C++
class Solution {
public:
    int maxArea(vector<int>& height) {
        if(height.empty()){
            return 0;
        }

        int l=0,r=height.size()-1;
        int ans = 0;
        while(l<r){
            int h = min(height[l],height[r]);
            ans = max(ans,(r-l)*h);
            if(height[l] < height[r]){
                l++;
            }else{
                r--;
            }
        }
        return ans;
    }
};
```

# 15. 三数之和
https://leetcode-cn.com/problems/3sum/

先排序，外层循环枚举，内层双指针取值。遇到数值相同的数字时，记得跳过。

```C++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        if(nums.empty()){
            return {};
        }
        sort(nums.begin(),nums.end());
        vector<vector<int>> ans;

        for(int i=0;i<nums.size();i++){
            if(nums[i] > 0){
                break;
            }

            if(i>0 && nums[i] == nums[i-1]){
                continue;
            }

            int n = nums[i];
            int l=i+1,r=nums.size()-1;
            while(l<r){
                if(n + nums[l] + nums[r] == 0){
                    ans.push_back({n,nums[l],nums[r]});
                    l++;r--;
                    while(l<r && nums[l] == nums[l-1]){
                        l++;
                    }
                    while(l<r && nums[r] == nums[r+1]){
                        r--;
                    }
                }else if(n + nums[l] + nums[r] < 0){
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

# 17. 电话号码的字母组合
https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

用一个map记录数字和字母的映射关系。然后取输入的数字，对每一个数字对应的字符，都做递归枚举回溯。

```C++
class Solution {
public:
    unordered_map<char,string> map;
    vector<string> ans;
    vector<string> letterCombinations(string digits) {
        if(digits.empty()){
            return {};
        }
        string path;
        init();
        backtrack(digits,path,0);
        return ans;


    }

    void backtrack(string& digits, string& path, int start){
        if(start >= digits.size()){
            ans.push_back(path);
            return ;
        }

        char c = digits[start];
        string s = map[c];
        for(auto t:s){
            path.push_back(t);
            backtrack(digits,path,start+1);
            path.pop_back();
        }
    }

    void init(){
        map['2'] = "abc";
        map['3'] = "def";
        map['4'] = "ghi";
        map['5'] = "jkl";
        map['6'] = "mno";
        map['7'] = "pqrs";
        map['8'] = "tuv";
        map['9'] = "wxyz";
    }
};
```

# 19. 删除链表的倒数第N个节点
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

链表快慢指针，让快指针先走n步，找到要删除的节点的前一个节点，让他的next等于next->next 。

注意判断如果快指针走快n步后已经到达末尾了，这时候说明要删的是第一个节点。返回head->next.

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
        if(head == NULL){
            return NULL;
        }

        ListNode* fast = head;
        ListNode* slow = head;

        while(n-- && fast){
            fast = fast->next;
        }

        if(fast == NULL){
            return head->next;
        }

        while(fast->next != NULL){
            fast = fast->next;
            slow = slow->next;
        }
        if(slow->next){
            slow->next = slow->next->next;
        }else{
            slow->next = NULL;
        }
        

        return head;

    }
};
```

# 20. 有效的括号
https://leetcode-cn.com/problems/valid-parentheses/

左括号入栈；右括号出栈。 注意出栈的时候先判断栈是不是空，如果是空，证明只有一个右括号，return false；

最后返回的时候也要判断栈是不是空，如果不是，证明有一个左括号没匹配，返回false。

```C++
class Solution {
public:
    bool isValid(string s) {
        if(s.empty()){
            return true;
        }

        stack<char> stack;

        for(auto c:s){
            if(c == '(' || c == '[' || c == '{'){
                stack.push(c);
            }else{
                if(stack.empty()){
                    return false;
                }
                char t = stack.top();
                if(c == ')'){
                    if(t != '('){
                        return false;
                    }else{
                        stack.pop();
                    }

                }else if(c == ']'){
                    if(t != '['){
                        return false;
                    }else{
                        stack.pop();
                    }
                }else if(c == '}'){
                    if(t != '{'){
                        return false;
                    }else{
                        stack.pop();
                    }
                }
            }
        }
        return stack.empty()?true:false;
    }
};
```

# 21. 合并两个有序链表
https://leetcode-cn.com/problems/merge-two-sorted-lists/

归并排序。

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
        else if(l2 == nullptr){
            return l1;
        }else if(l1 == nullptr && l2 == nullptr){
            return nullptr;
        }
        ListNode* dummy = new ListNode(-1);
        ListNode* current = dummy;
        while(l1 && l2){
            int val1 = l1->val;
            int val2 = l2->val;
            if(val1<val2){
                current->next = new ListNode(val1);
                current = current->next;
                l1 = l1->next;
            }else{
                current->next = new ListNode(val2);
                current = current->next;
                l2 = l2->next;
            }
        }

        while(l1){
            current->next = new ListNode(l1->val);
            current = current->next;
            l1 = l1->next;
        }
        while(l2){
            current->next = new ListNode(l2->val);
            current = current->next;
            l2 = l2->next;
        }
        return dummy->next;
    }
};
```

# 22. 括号生成
https://leetcode-cn.com/problems/generate-parentheses/

维护两个变量记录可用的左右括号的数量；然后从可以用的括号数量开始做选择，回溯；每次选一次左括号或右括号，递归的时候对应括号计数减1；当左右括号数量刚好为0时，证明生成的括号合符要求。

```C++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> ans;
        string path;
        backtrack(ans,path,n,n,n);
        return ans;
    }

    void backtrack(vector<string>& ans, string& path, int n, int left, int right){
        if(left < 0 || right < 0){
            return ;
        }
        if(left > right){
            return ;
        }
        if(left == 0 && right == 0){
            ans.push_back(path);
            return ;
        }

        path.push_back('(');
        backtrack(ans,path,n,left-1,right);
        path.pop_back();

        path.push_back(')');
        backtrack(ans,path,n,left,right-1);
        path.pop_back();

    }
};
```

# 23. 合并K个升序链表
https://leetcode-cn.com/problems/merge-k-sorted-lists/

递归归并排序。每次取vector中间开始二分做递归；递归到最后是两个list；对这两个list进行归并排序。

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
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if(lists.empty()){
            return nullptr;
        }
        return merge(lists,0,lists.size()-1);
    }

    ListNode* merge(vector<ListNode*> &lists, int left, int right){
        if(left == right){
            return lists[left];
        }
        int mid = left+(right-left)/2;
        ListNode* l1 = merge(lists,left,mid);
        ListNode* l2 = merge(lists,mid+1,right);
        return mergeTwoLists(l1,l2);
    }

    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2){
        if(l1 == nullptr){
            return l2;
        }
        if(l2 == nullptr){
            return l1;
        }
        ListNode* dummy = new ListNode(-1);
        ListNode* current = dummy;
        while(l1 && l2){
            if(l1->val < l2->val){
                current->next = l1;
                l1 = l1->next;
                current = current->next;
            }else{
                current->next = l2;
                l2 = l2->next;
                current = current->next;
            }
        }
        while(l1){
            current->next = l1;
            l1 = l1->next;
            current = current->next;
        }
        while(l2){
            current->next = l2;
            l2 = l2->next;
            current = current->next;
        }
        return dummy->next;
    }
};
```

# 31. 下一个排列
https://leetcode-cn.com/problems/next-permutation/

从右边找降序序列里面第一个遇到的升序的元素，那个元素就是后面要和比他大但是和他最接近的元素交换位置的元素，我们可以叫元素A。

如果找到他的位置是在头部，说明这时候全是降序，下一个排列就是升序序列。

然后我们要找比他大但是和他最接近的元素B，和他交换位置；也是从右边开始找，如果当前找的元素比他小，就继续找；找到以后，将元素A和元素B进行交换，交换以后，从元素B后对剩下的元素继续进行排序，就是下一个序列了。

考虑序列[0,1,4,3,2]、找到要交换的元素A(1)，找到最接近1的元素(2)，将1和2交换、[0,2,4,3,1]；从2以后对序列排列[0,2,1,3,4]。

```C++
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int i=nums.size()-1;
        while(i>0 && nums[i-1] >= nums[i]){
            i--;
        }
        if(i == 0){
            reverse(nums.begin(),nums.end());
        }else{
            int j=nums.size()-1;
            while(j>i && nums[j] <= nums[i-1]){
                j--;
            }
            swap(nums[i-1],nums[j]);
            sort(nums.begin()+i,nums.end());
        }
    }
};
```
# 33. 搜索旋转排序数组
https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

logn时间复杂度，首先想到就是二分；通过判断mid和左右边界元素的大小，知道某一部分是有序的；然后在有序的部分进行二分；注意边界条件的判断，比如nums[mid]< target ，除此之外还要target<=nums[r] ，否则要找的元素还是在另外一侧。

```C++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        return search(nums,target,0,nums.size()-1);
    }

    int search(vector<int>& nums, int target ,int l, int r){
        while(l<=r){
            int mid = l+(r-l)/2;
            if(nums[mid] == target){
                return mid;
            }
            //mid-r in order
            else if(nums[mid] < nums[r]){
                if(nums[mid] < target && target <= nums[r]){
                    l=mid+1;
                }else{
                    r=mid-1;
                }
            }
            //l-mid in order
            else{
                if(target < nums[mid] && target >= nums[l]){
                    r=mid-1;
                }else{
                    l=mid+1;
                }
            }
        }
        
        return -1;
    }
};
```

# 34. 在排序数组中查找元素的第一个和最后一个位置
https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/

logn时间复杂度，有序数组，典型二分。

二分找到mid后，对其进行中心扩展，找到相同的数值的左右边界，构造数组返回。

```C++
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> ans;
        int l=0,r=nums.size()-1;

        while(l<=r){
            int mid = l+(r-l)/2;
            if(nums[mid] == target){
                int m1 = mid, m2 = mid;
                while(m1 > 0 && nums[m1] == nums[m1-1]){
                    m1--;
                }

                while(m2 < nums.size()-1 && nums[m2] == nums[m2+1]){
                    m2++;
                }
                ans.push_back(m1);
                ans.push_back(m2);
                break;
                
            }else if(nums[mid] < target){
                l=mid+1;
            }else if(nums[mid] > target){
                r = mid-1;
            }
        }


        return ans.empty()?vector<int>{-1,-1}:ans;
    }
};
```

# 39. 组合总和
https://leetcode-cn.com/problems/combination-sum/

排序，回溯；当sum==target时结束递归回溯。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        if(candidates.empty()){
            return {};
        }
        sort(candidates.begin(),candidates.end());
        backtrack(candidates,0,0,target);
        return ans;

    }

    void backtrack(vector<int>& candidates, int start, int sum, int target){
        if(start >= target){
            return ;
        }
        if(sum == target){
            ans.push_back(path);
            return ;
        }

        for(int i=start;i<candidates.size();i++){
            if(sum > target){
                break;
            }
            sum += candidates[i];
            path.push_back(candidates[i]);
            backtrack(candidates,i,sum,target);
            sum -= candidates[i];
            path.pop_back();
        }
    }
};
```

# 42. 接雨水
https://leetcode-cn.com/problems/trapping-rain-water/

双指针，选择两边较矮的柱子；如果当前较矮的柱子比其对应边上的最高柱子要高，就更新对应侧最高柱子的值；如果比最高柱子要矮，就可以更新接雨水的雨量了。

```C++
class Solution {
public:
    int trap(vector<int>& height) {
        if(height.empty()){
            return 0;
        }

        int leftMaxHeight = 0, rightMaxHeight = 0;

        int i=0,j=height.size()-1;
        int ans = 0;

        while(i<j){
            if(height[i] < height[j]){
                if(height[i] > leftMaxHeight){
                    leftMaxHeight = height[i];
                }else{
                    ans += leftMaxHeight - height[i];
                }
                i++;
            }else{
                if(height[j] > rightMaxHeight){
                    rightMaxHeight = height[j];
                }else{
                    ans += rightMaxHeight - height[j];
                }
                j--;
            }
        }
        return ans;
    }
};
```

# 46. 全排列
https://leetcode-cn.com/problems/permutations/

排列问题，回溯+visited数组跳过重复数字。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;

    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> visisted(nums.size(),false);
        backtrack(nums,visisted);
        return ans;
    }

    void backtrack(vector<int>& nums, vector<bool>& visisted){
        if(path.size() == nums.size()){
            ans.push_back(path);
            return;
        }

        for(int i=0;i<nums.size();i++){
            if(visisted[i]){
                continue;
            }

            visisted[i] = true;
            path.push_back(nums[i]);
            backtrack(nums,visisted);
            visisted[i] = false;
            path.pop_back();
        }
    }
};
```

# 48. 旋转图像
https://leetcode-cn.com/problems/rotate-image/

顺时针，交换 左上-右下 对角线对称的元素，然后每一行交换中点对称元素；
逆时针，交换 右上-左下 对角线对称的元素，然后每一行交换中点对称元素。

```C++
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        if(matrix.empty()){
            return ;
        }

        for(int i=0;i<matrix.size();i++){
            for(int j=0;j<i;j++){
                swap(matrix[i][j],matrix[j][i]);
            }
        }

        for(int i=0;i<matrix.size();i++){
            for(int j=0;j<matrix[i].size()/2;j++){
                swap(matrix[i][j],matrix[i][matrix[i].size() - 1 - j]);
            }
        }
    }
};
```

# 49. 字母异位词分组
https://leetcode-cn.com/problems/group-anagrams/

对每个单词进行排序，然后将其排序后的结果作为map的key；value就是各种字符相同的异位词分组。

然后遍历一下map，就可以拿到每个key对应的分组了。

```C++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        vector<vector<string>> ans;
        map<string,vector<string>> map;

        for(auto &s:strs){
            string key = s;
            sort(key.begin(),key.end());
            map[key].push_back(s);
        }

        for(auto it = map.begin();it!=map.end();it++){
            ans.push_back(it->second);
        }
        return ans;
    }
};
```

# 53. 最大子序和
https://leetcode-cn.com/problems/maximum-subarray/

经典dp。用一个dp表示当前的子序和；

对每一位做选择，不选的话dp的值就是nums[i]，选了的话dp的值就是dp[i-1]+nums[i]。

然后对dp取它的最大值就可以了。

```C++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }
        int ans = nums[0];
        vector<int> dp(nums.size(),0);
        dp[0] = nums[0];
        for(int i=1;i<nums.size();i++){
            dp[i] = max(dp[i-1]+nums[i],nums[i]);
            ans = max(ans,dp[i]);
        }
        return ans;
    }
};
```

# 55. 跳跃游戏
https://leetcode-cn.com/problems/jump-game/

贪心。维护一个可以到达最长距离的变量初始化为第0号做标的值，然后从1开始遍历数组，直到倒数第二位。

每次遍历坐标时，先判断一下当前位置可不可达（i<=distance），如果可以到达，我们更新一下可以到达的最长距离；否则就不更新。

最后判断一下可以到达的最长距离是否超过数组长度后一位就可以了。

```C++
```

# 56. 合并区间
https://leetcode-cn.com/problems/merge-intervals/

将区间按照左边界来排序；然后先将第一个区间放入ans。

然后后面每次都将当前区间和ans中最后一个区间进行比较，如果有重叠就更新ans中最后一个区间的右边界；否则直接push_back。

```C++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if(intervals.empty()){
            return {};
        }
        vector<vector<int>> ans;
        vector<int> temp;

        sort(intervals.begin(),intervals.end(),[](auto &a, auto &b){
            return a[0]==b[0]?a[1]<b[1]:a[0]<b[0];
        });
        
        ans.push_back(intervals[0]);
        for(int i=1;i<intervals.size();i++){
            //(1,4)(2,5)
            if(intervals[i][0] <= ans.back()[1]){
                ans.back()[1] = max(intervals[i][1],ans.back()[1]);
            }else{
                //(1,4)(5,6)
                ans.push_back({intervals[i][0],intervals[i][1]});
            }
        }

        return ans;
    }
};
```

# 62. 不同路径
https://leetcode-cn.com/problems/unique-paths/

一开始觉得是用回溯去做，结果超时了。。看了评论才发现和之前的走楼梯是差不多的，只不过走楼梯是一维动态规划，这里是二维。注意基础状态（第一行和第一列）都是1 。

```C++
class Solution {
public:
    int ans = 0;
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m,vector<int>(n,1));
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};
```

# 64. 最小路径和
https://leetcode-cn.com/problems/minimum-path-sum/

经典二维dp。

```C++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        if(grid.empty()){
            return 0;
        }

        vector<vector<int>> dp(grid.size(),vector<int>(grid[0].size(),0));

        dp[0][0] = grid[0][0];
        for(int i=1;i<grid.size();i++){
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }

        for(int j=1;j<grid[0].size();j++){
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }

        for(int i=1;i<grid.size();i++){
            for(int j=1;j<grid[0].size();j++){
                dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j];
            }
        }

        return dp[grid.size()-1][grid[0].size()-1];
    }
};
```

# 581. 最短无序连续子数组
https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray/

直观的就是借助一个排好序的数组，然后双指针，找到左右第一个和排好序后不一样的元素的index。
长度就是j-i+1.

```C++
class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }
        vector<int> temp(nums.begin(),nums.end());
        sort(temp.begin(),temp.end());

        int i=0,j=nums.size()-1;

        while(i<=j && nums[i] == temp[i]){
            i++;
        }
        while(i<=j && nums[j] == temp[j]){
            j--;
        }

        return j-i+1;
    }
}
```

# 647. 回文子串
https://leetcode-cn.com/problems/palindromic-substrings/

中心扩散法，枚举每个字符，作为回文串的中心字符；

然后从中间开始（分奇数和偶数字符串长度计算），分别判断是否回文串，进行计算。

```C++
class Solution {
public:
    int countSubstrings(string s) {
        int n = s.size();
        int ans = n;
        for(int i = 1 ; i < n ; ++i){
            int l = i, r = i - 1;
            while(--l >= 0 && ++r < n && s[l] == s[r])  ans++;
            l = i, r = i;
            while(--l >= 0 && ++r < n && s[l] == s[r])  ans++;
            }
        return  ans;
    }
};
```
