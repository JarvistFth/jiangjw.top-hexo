---
title: leetcode-hot100-2
date: 2020-09-22 19:13:30
categories:
- leetcode
- hot100
tags: [leetcode ,hot100]
keywords: [leetcode,hot100]
---
leetcode hot100 合集，第二部分。
<!---more--->

## 70. 爬楼梯
https://leetcode-cn.com/problems/climbing-stairs/

经典dp。dp[n]表示到n层的爬法个数；到第i层的爬法，我们可以从i-1层爬1层，或者从i-2层爬2层。所以dp[i]=dp[i-1]+dp[i-2]。

初始状况dp[1]=1,dp[2]=2 。dp[n]是所求。

```C++
class Solution {
public:
    int times = 0;
    int climbStairs(int n) {
        
        if(n<=2){
            return n;
        }
        int *dp = new int [n+1];
        dp[1] = 1;
        dp[2] = 2;
        for(int i=3;i<=n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
};
```

## 72. 编辑距离
https://leetcode-cn.com/problems/edit-distance/

知道是dp。。但是不会构造dp。。评论的题解很棒。。。嗯。。。

dp[i][j]表示word1[0...i]变成word2[0...j]的最小操作数。

考虑：word1[0...i] word2[0...j]和word1[i],word2[j]；

1. 如果word1[i]==word2[j]，操作数是dp[i-1][j-1]；因为当前字符相同，不用操作；
2. 否则需要操作。操作有三种，改变、增加、删除；
3. 如果是改变，就是认为word1[0...i-1] == word2[0...j-1]，dp[i-1][j-1]的操作数上+1，将word1[i]变成word2[j]；
4. 如果是删除，就是认为word1[0...i-1] == word2[0...j]，需要在word1[0...i]上删除word1[i]，也是一步操作。而word1[0...i-1]->word2[0...j] = dp[i-1][j]。
5. 同理，如果是增加，就是认为word1[0...i] == word2[0...j-1],需要在word1[0...j-1]上增加一个word2[j]，也是一步操作。而word1[0...i]->word2[0...j-1] = dp[i-1][j]。

综上：dp[i][j] = 1+min({dp[i-1][j-1],dp[i-1][j],dp[i][j-1]})。

```C++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size();
        int n = word2.size();

        vector<vector<int>> dp(m+1,vector<int>(n+1));

        for(int i=0;i<=m;i++){
            dp[i][0] = i;
        }
        for(int j=0;j<=n;j++){
            dp[0][j] = j;
        }

        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(word1[i-1] == word2[j-1]){
                    dp[i][j] = dp[i-1][j-1];
                }else{
                    dp[i][j] = 1+min({dp[i-1][j-1],dp[i-1][j],dp[i][j-1]});
                }
            }
        }
        return dp[m][n];
    }
};
```

## 75. 颜色分类
https://leetcode-cn.com/problems/sort-colors/

双指针指向0和2数字的边界，然后再维护一个指针指向当前元素；

遍历到的元素如果是0，就和左边界的元素交换；如果是2，就和右边界的元素交换；如果是1，就直接继续遍历。

这样就可以维护一个左边是0，右边是2，中间是1的数组了。

```C++
class Solution {
public:
    void sortColors(vector<int>& nums) {
        if(nums.empty()){
            return;
        }

        int l=0,r=nums.size()-1;
        int i=0;
        while(i<=r){
            if(nums[i] == 0){
                nums[i] = nums[l];
                nums[l] = 0;
                i++;l++;
            }else if(nums[i] == 1){
                i++;
            }else if(nums[i] == 2){
                nums[i] = nums[r];
                nums[r] = 2;
                r--;
            }
        }
        return ;
    }
};
```

## 76. 最小覆盖子串
https://leetcode-cn.com/problems/minimum-window-substring/

滑动窗口，一个维护当前窗口，一个维护需要的子串字符。

维护一个valid变量，表示子串所需的字符数；当我们所需的字符数符合子串字符数时，开始出窗；

出窗的时候也要判断出窗的字符是不是所需字符，如果是，就要更新valid变量；

出窗开始前，确定当前字符串长度，如果当前子串长度小于上次的长度，就更新长度。

```C++
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char,int> window;
        unordered_map<char,int> need;

        int start=0,len=INT_MAX;
        int l=0,r=0;
        int valid = 0;

        for(auto c:t){
            need[c]++;
        }

        while(r<s.size()){
            char c = s[r];
            r++;
            if(need.count(c)){
                window[c]++;
                if(window[c] == need[c]){
                    valid++;
                }
            }

            while(valid == need.size()){
                if(r-l < len){
                    start = l;
                    len = r-l;
                }
                char d = s[l];
                l++;
                if(need.count(d)){
                    if(window[d] == need[d]){
                        valid--;
                    }
                    window[d]--;
                }
            }
        }
        return len==INT_MAX?"":s.substr(start,len);

    }
};
```

## 78. 子集
https://leetcode-cn.com/problems/subsets/

回溯。
子集问题的时候，每次开始回溯的时候，都将当前的path加入到ans中。维护一个start，跳过之前遍历过的元素。

```C++
class Solution {
public:
    vector<vector<int>> ans;
    // vector<int> path;
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<int> path;
        backtrack(nums,0,path);
        return ans;
    }

    void backtrack(vector<int>& nums, int start, vector<int>& path){
        ans.push_back(path);

        for(int i=start;i<nums.size();i++){
            path.push_back(nums[i]);
            backtrack(nums,i+1,path);
            path.pop_back();
        }
        
    }
};
```

## 79. 单词搜索
https://leetcode-cn.com/problems/word-search/

迷宫类dfs搜索。对每个位置都要进行一次dfs搜索。

```C++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(board.size() == 0){
            return false;
        }

        // vector<vector<bool>> visited(board.size(),vector<bool>(board[0].size(),false));

        bool ans = false;

        int index = 0;

        for(int i=0;i<board.size();i++){
            for(int j=0;j<board[i].size();j++){
                ans = ans || dfs(board,word,index,i,j);
            }
        }
        return ans;
    }

    bool dfs(vector<vector<char>>& board, string& word, int index, int x, int y){

        if(index >= word.size()){
            return true;
        }

        if(x<0 || x>=board.size() || y<0 || y>=board[0].size() || board[x][y]=='0' || board[x][y]!=word[index]){
            return false;
        }

        char temp = board[x][y];
        board[x][y] = '0';

        if(dfs(board,word,index+1,x+1,y) || dfs(board,word,index+1,x-1,y) ||
            dfs(board,word,index+1,x,y+1) || dfs(board,word,index+1,x,y-1)){
                return true;
        }
        board[x][y] = temp;
        return false;
    }
};
```

## 84. 柱状图中最大的矩形
https://leetcode-cn.com/problems/largest-rectangle-in-histogram/

单调增栈。

栈顶的柱子是最高的，其形成最大的矩形是由其左右两侧比他小的矩形来确定的。其左边的柱子是栈顶元素的下一个元素；右侧元素就是单调增栈需要入栈时的那个元素。

前加0，防止pop后，s.top()为空；后加0，防止右边界是最大值不更新ans。

```C++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int> s;
        vector<int> temp(heights.size()+2);
        //前加0，防止pop后，s.top()为空；后加0，防止右边界是最大值不更新ans。
        copy(heights.begin(),heights.end(),temp.begin()+1);
        int ans = 0;
        for(int i = 0;i<temp.size();i++){
            while(!s.empty() && temp[i] < temp[s.top()]){
                int h = temp[s.top()];
                s.pop();
                ans = max(ans,(i-s.top()-1) * h);
            }
            s.push(i);
        }
        return ans;
    }
};
```

## 85. 最大矩形

这题可以看作是上一题的延伸，对每一行都进行一次求最大矩形；用一个vector表示当前位置的柱形图，如果matrix的值是1，就让柱形图++；否则归0。

---
每一层看作是柱状图，可以套用84题柱状图的最大面积。

第一层柱状图的高度["1","0","1","0","0"]，最大面积为1；

第二层柱状图的高度["2","0","2","1","1"]，最大面积为3；

第三层柱状图的高度["3","1","3","2","2"]，最大面积为6；

第四层柱状图的高度["4","0","0","3","0"]，最大面积为4；

```C++
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        if(matrix.empty()){
            return 0;
        }
        int ans = 0;
        vector<int> heights(matrix[0].size()+2);
        for(int i=0;i<matrix.size();i++){
            for(int j=0;j<matrix[i].size();j++){
                if(matrix[i][j] == '1'){
                    heights[j+1] += 1;
                }else{
                    heights[j+1] = 0;
                }
            }
            ans = max(ans,largestRectangleArea(heights));
        }
        return ans;
    }


    int largestRectangleArea(vector<int>& heights) {
        stack<int> s;
        // vector<int> temp(heights.size()+2);
        //前加0，防止pop后，s.top()为空；后加0，防止右边界是最大值不更新ans。
        // copy(heights.begin(),heights.end(),temp.begin()+1);
        int ans = 0;
        for(int i = 0;i<heights.size();i++){
            while(!s.empty() && heights[i] < heights[s.top()]){
                int h = heights[s.top()];
                s.pop();
                ans = max(ans,(i-s.top()-1) * h);
            }
            s.push(i);
        }
        return ans;
    }
};
```

## 139. 单词拆分
https://leetcode-cn.com/problems/word-break/

类似背包问题的dp。dp[i]表示string的前i个字符能否被拆分；如果我们假设前j个字符已经可以被成功拆分，那么只需要看j-i个字符是否在dict中就可以了。

```C++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool> dp(s.size()+1,false);
        unordered_set<string> dict(wordDict.begin(),wordDict.end());

        dp[0] = true;

        for(int i=1;i<=s.size();i++){
            for(int j=0;j<i;j++){
                if(dp[j] && dict.count(s.substr(j,i))){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```

## 94. 二叉树的中序遍历
https://leetcode-cn.com/problems/binary-tree-inorder-traversal/

```C++
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
    vector<int> ans;
    vector<int> inorderTraversal(TreeNode* root) {
        if(root == nullptr){
            return {};
        }

        stack<TreeNode*> nodeStack;
        TreeNode* node = root;
        
        while(node || !nodeStack.empty()){
            if(node){
                nodeStack.push(node);
                node = node->left;
            }else{
                node = nodeStack.top();
                nodeStack.pop();
                ans.push_back(node->val);
                node = node->right;
            }
        }
        return ans;
    }
};
```

## 96. 不同的二叉搜索树
https://leetcode-cn.com/problems/unique-binary-search-trees/

对于n个节点的树，其左子树的数量可以是0,1,2,...n-1；右子树对应的就是n-1,n-2...0;
假设n个节点存在二叉排序树的个数是G(n)，令f(i)为以i为根的二叉搜索树的个数
即有:

G(n) = f(1) + f(2) + f(3) + f(4) + ... + f(n)

所以当i为根节点时，其左子树节点个数为i-1个，右子树节点为n-i，即f(i) = G(i-1)*G(n-i),


```C++
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n+1,0);
        dp[0]=1;
        for(int i=1;i<=n;i++){
            for(int j=1;j<=i;j++){
                dp[i] += dp[j-1]*dp[i-j];
            }
        }
        return dp[n];
    }
};
```

## 98. 验证二叉搜索树
https://leetcode-cn.com/problems/validate-binary-search-tree/

中序遍历，用一个pre记录前一个遍历的节点，判断他和当前节点的大小；在中序遍历的时候更新pre。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* pre = NULL;
    bool isValidBST(TreeNode* root) {
        if(root == NULL){
            return true;
        }
        else{
            if(!isValidBST(root->left)){
                return false;
            }
            if(pre != NULL && pre->val>=root->val){
                return false;
            }
            pre = root;
            if(!isValidBST(root->right)){
                return false;
            }
            return true;
        }
        
    }
};
```

## 101. 对称二叉树
https://leetcode-cn.com/problems/symmetric-tree/

判断二叉树的数值是否相同或者某一子树是否为空，注意对称的要求，递归的时候传入的两个节点一个是left一个是right就可以。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        return isSymmetric(root,root);
    }

    bool isSymmetric(TreeNode* root1, TreeNode* root2){
        if(!root1 && !root2){
            return true;
        }
        if(!root1 || !root2){
            return false;
        }
        if(root1->val != root2->val){
            return false;
        }
        return isSymmetric(root1->left,root2->right) && isSymmetric(root1->right,root2->left); 
    }
};
```

## 104. 二叉树的最大深度
https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/

递归，每层+1返回。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        return 1+max(maxDepth(root->left),maxDepth(root->right));
    }
};
```

## 105. 从前序与中序遍历序列构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

递归建树，每次传入左子树范围和右子树的范围。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return buildTree(preorder,inorder,0,preorder.size()-1,0,inorder.size()-1);
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder, int preleft, int preright, int inleft, int inright){
        if(preleft>preright || inleft>inright){
            return NULL;
        }

        int rootval = preorder[preleft];
        TreeNode* root = new TreeNode(rootval);
        int index = 0;
        for(int i=0;i<=inright;i++){
            if(inorder[i] == rootval){
                index = i;
                break;
            }
        }
        root->left = buildTree(preorder,inorder,preleft+1,preleft+index-inleft,inleft,index-1);
        root->right = buildTree(preorder,inorder,preleft+1+index-inleft,preright,index+1,inright);
        return root;
    }
};
```

## 114. 二叉树展开为链表
https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/

递归将左右子树捋直了，然后将左子树接到右子树上。

```C++
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
    void flatten(TreeNode* root) {
        if(root == nullptr){
            return ;
        }

        flatten(root->right);
        flatten(root->left);
        
        auto tmp = root->right;
        root->right = root->left;
        root->left = nullptr;
        while(root->right){
            root = root->right;
        }
        root->right = tmp;
    }
};
```

## 124. 二叉树中的最大路径和
https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/

递归获取左右子树的最大路径和（如果是负数就让他为0）

向上返回左右子树的最大值+当前root的值；最大路径和可能是root的左右子树构成的路径。

 ans=max(ans，(left+right+root->val))。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    int ans = INT_MIN;
    int maxPathSum(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        dfs(root);
        return ans;
    }

    int dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        auto left = max(0,dfs(root->left));
        auto right = max(0,dfs(root->right));
        ans = max(ans,left+right+root->val);
        return max(left,right)+root->val;
    }
};
```

## 128. 最长连续序列
https://leetcode-cn.com/problems/longest-consecutive-sequence/

用哈希表存储每个端点值对应连续区间的长度

若数已在哈希表中：跳过不做处理

若是新数加入：

1. 取出其左右相邻数已有的连续区间长度 left 和 right
2. 计算当前数的区间长度为：cur_length = left + right + 1
3. 根据 cur_length 更新最大长度 max_length 的值
4. 更新区间两端点的长度值

```C++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if(nums.empty()){
            return 0;
        }
        int ans = 0;

        unordered_map<int,int> map;
        int left=0,right=0;
        for(auto n:nums){
            int currentLength = 0;
            if(map[n] == 0){
                left = map[n-1];
                right = map[n+1];
                currentLength = 1+left+right;

                ans = max(ans,currentLength);
                map[n] = currentLength;
                map[n-left] = currentLength;
                map[n+right] = currentLength;
            }
        }
        return ans;
    }
};
```

## 136. 只出现一次的数字
https://leetcode-cn.com/problems/single-number/

每个数字和自己异或会变成0.

```C++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ans = 0;
        for(auto n:nums){
            ans ^= n;
        }
        return ans;
    }
};
```

## 139. 单词拆分
https://leetcode-cn.com/problems/word-break/

背包模型dp；dp[j]表示第j个单词可以被划分；那么[j,...,i]的单词能不能被划分取决于dict中是否有s[j,...i]。

```C++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool> dp(s.size()+1,false);
        unordered_set<string> dict(wordDict.begin(),wordDict.end());

        dp[0] = true;

        for(int i=1;i<=s.size();i++){
            for(int j=0;j<i;j++){
                if(dp[j] && dict.count(s.substr(j,i-j))){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```

## 141. 环形链表
https://leetcode-cn.com/problems/linked-list-cycle/

快慢指针，如果有环，快指针和慢指针最终会遇上（相等）。

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

## 146. LRU缓存机制
https://leetcode-cn.com/problems/lru-cache/

哈希双向链表；通过哈希O(1)找到key对应的listnode是否存在；

通过双向链表，增删节点；

1. 如果此时key/value已经存在，将其删除，然后往链表头插入；
2. 如果此时key/value不存在，新建个节点；如果这时候已经超出容量了，删除尾部节点再往头部插入；否则直接往头部插入。
3. 获取key/value的时候，直接通过哈希表的key获取对应的node节点；然后再将其put进LRUCache就可以了。

```C++
class LRUCache {
public:
    LRUCache(int capacity) : capacity(capacity),size(0) {
        head = new ListNode();
        tail = new ListNode();
        head->next = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if(map.count(key) == 0){
            return -1;
        }
        int val = map[key]->val;
        put(key,val);
        return val;
    }
    
    void put(int key, int value) {
        ListNode* node = new ListNode(key,value);
        //key exist 
        
        if(map.count(key)){
            removeOne(map[key]);
            addFirst(node);
            map[key]=node;
        }else{
            if(capacity == size){
                auto back = removeLast();
                map.erase(back->key);
                delete back;
                size--;
            }
            addFirst(node);
            map[key] = node;
            size++;
        }
    }

    

private:
    struct ListNode{
        int key;
        int val;
        ListNode* prev;
        ListNode* next;
        ListNode():key(0),val(0),prev(nullptr),next(nullptr){};
        ListNode(int key,int val):key(key),val(val),prev(nullptr),next(nullptr){};
    };
    unordered_map<int,ListNode*>map;
    ListNode* head;
    ListNode* tail;
    int size;
    int capacity;


    void addFirst(ListNode* node){
        auto n = head->next;
        head->next = node;
        node->next = n;
        n->prev = node;
        node->prev = head;
    }

    ListNode* removeLast(){
        auto n = tail->prev;
        removeOne(n);
        return n;
    }

    void removeOne(ListNode* node){
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
 ```

## 148. 排序链表
https://leetcode-cn.com/problems/sort-list/

归并或者快排版本都有。

1. 归并排序先快慢指针找中点，然后断链，递归调用断链；对左右两部分进行归并排序；
2. 快排

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
    ListNode* sortList(ListNode* head) {
        if(head == NULL){
            return NULL;
        }
        // ListNode* end = head;
        // while(end->next){
        //     end = end->next;
        // }
        // quickSort(head,end);
        // return head;
        return mergeSort(head);

    }

    // void quickSort(ListNode *start, ListNode *end){
    //     if(start == NULL || end == NULL || start == end){
    //         return ;
    //     }

    //     ListNode* p = start;
    //     ListNode* q = start->next;
    //     int midval = start->val;
    //     while(q != end->next){
    //         if(q->val < midval){
    //             p = p->next;
    //             if(p != q){
    //                 swap(p->val,q->val);
    //             }
    //         }
    //         q = q->next;
    //     }
    //     if(p != start){
    //         swap(p->val ,start->val);
    //     }
    //     quickSort(start,p);
    //     quickSort(p->next,end);
    // }

    

    ListNode* mergeSort(ListNode* head){
        if(!head || !head->next){
            return head;
        }
        ListNode* fast = head;
        ListNode* slow = head;
        ListNode* pre = head;

        while(fast  && fast->next ){
            fast = fast->next->next;
            pre = slow;
            slow = slow->next;
        }
        pre->next = NULL;

        ListNode* l = mergeSort(head);
        ListNode* r = mergeSort(slow);
        return merge(l,r);
    }

    ListNode* merge(ListNode* l1, ListNode* l2){
        ListNode* dummy = new ListNode(-1);
        ListNode* current = dummy;

        while(l1!=NULL && l2 != NULL){
            if(l1->val <= l2->val){
                current->next = l1;
                current = current->next;
                l1 = l1->next;
            }else{
                current->next = l2;
                current = current->next;
                l2 = l2->next;
            }
        }

        while(l1 != NULL){
            current->next = l1;
            current = current->next;
            l1 = l1->next;
        }
        while(l2 != NULL){
            current->next = l2;
            current = current->next;
            l2 = l2->next;
        }
        return dummy->next;
    }
};
```

## 152. 乘积最大子数组
https://leetcode-cn.com/problems/maximum-product-subarray/

遇到负数会让最大值变最小值，最小值变最大值；所以在更新最大值和最小值前，先判断一下当前元素是不是负数；如果是负数就先把他们交换。

```C++
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        if(nums.size() == 0 ){
            return 0;
        }

        int ans = INT_MIN;

        int maxval = 1,minval = 1;
        for(auto n:nums){
            if(n<0){
                swap(maxval,minval);
            }
            maxval = max(maxval * n,n);
            minval = min(minval * n, n);
            ans = max(maxval,ans);
        }
        return ans;
    }
};
```

## 155. 最小栈
https://leetcode-cn.com/problems/min-stack/

两个栈，一个正常出入，一个只将最小值入栈；如果当前push进来的值小于top的值，就将top push进去。getMin其实就是获取最小值的栈的top。

```C++
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int> s;
    stack<int> v;
    MinStack() {
        
    }
    
    void push(int x) {
        s.push(x);
        if(v.empty()){
            v.push(x);
        }else{
            if(x < v.top()){
                v.push(x);
            }else{
                v.push(v.top());
            }
        }
    }
    
    void pop() {
        if(s.empty() || v.empty()){
            return ;
        }
        s.pop();
        v.pop();
    }
    
    int top() {
        return s.empty()?0:s.top();
    }
    
    int getMin() {
        return v.empty()?0:v.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->getMin();
 */
 ```

## 160. 相交链表
https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

每次走到一个本链表的末尾就让指针指向另一个链表的头；这样走两次后，如果有环，环的起点前走过的长度相等，此时pA==pB。返回pA即可。

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
        if(headA == NULL || headB == NULL){
            return NULL;
        }
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

## 169. 多数元素
https://leetcode-cn.com/problems/majority-element/

哈希表是最容易想到的。O(1)空间用摩尔投票法，每次如果遇到相同的数，就count++，否则count--；当count为0的时候，就换一个数去统计。最后剩下的肯定是count不为0的数，就是答案。

```C++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        // if(nums.empty()){
        //     return 0;
        // }
        // map<int,int> map;
        // for(auto n:nums){
        //     map[n]++;
        // }
        // for(auto&[k,v] : map){
        //     if(v > nums.size()/2){
        //         return k;
        //     }
        // }
        // return 0;
        int count = 0;
        int ans = 0;
        for(auto n:nums){
            if(count == 0){
                ans = n;
                count++;
            }else{
                ans == n?count++:count--;
            }
        }
        return ans;
    }
};
```

## 347. 前 K 个高频元素
https://leetcode-cn.com/problems/top-k-frequent-elements/

map记录元素的出现次数，然后按照出现次数开始建小根堆；然后将前K个元素入堆；

如果当前堆的大小为K：从K+1个元素开始，如果当前元素大于堆顶的元素，pop堆顶；将该元素入堆；否则的话，丢弃当前元素。

如果不为K，直接入堆。

这样堆中保留的，就是前K个高频元素。

```C++
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int> map;
        for(auto &n:nums){
            map[n]++;
        }
        auto cmp = [](auto &a, auto &b){
            return a.second>b.second;
        };
        priority_queue<pair<int,int>,vector<pair<int,int>>,decltype(cmp)> queue(cmp);

        for(auto& [val,times]: map){
            if(queue.size() == k){
                if(times > queue.top().second){
                    queue.pop();
                    queue.emplace(val,times);
                }
            }else{
                queue.emplace(val,times);
            }
        }

        vector<int> ans;
        while(!queue.empty()){
            ans.push_back(queue.top().first);
            queue.pop();
        }
        return ans;

    }
};
```