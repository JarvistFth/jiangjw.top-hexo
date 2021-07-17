---
title: 【树】 - 二叉搜索树
date: 2020-06-25 17:22:20
categories: 
- leetcode
- 树
tags: [leetcode,树]
---

二叉搜索树，看到二叉搜索树首先就要想到中序遍历。一般都是采用中序遍历来做的。
<!---more--->

## 98. 验证二叉搜索树
https://leetcode-cn.com/problems/validate-binary-search-tree/

https://leetcode-cn.com/problems/legal-binary-search-tree-lcci/

维护一个prev节点作为遍历的前一个节点，这样我们在中序遍历的时候，在处理当前节点时，就可以和前一个节点的值做比较。

只要当前节点的值小于前一个节点的值，这个就不是二叉搜索树。

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
    //维护一个prev节点作为遍历的前一个节点
    TreeNode* pre = NULL;
    bool isValidBST(TreeNode* root) {
        if(root == NULL){
            return true;
        }
        else{
            if(!isValidBST(root->left)){
                return false;
            }
            //prev是空的话证明是第一个节点
            if(pre != NULL && pre->val>=root->val){
                return false;
            }
            //更新prev
            pre = root;
            if(!isValidBST(root->right)){
                return false;
            }
            return true;
        }
        
    }
};
```

## 96. 不同的二叉搜索树
https://leetcode-cn.com/problems/unique-binary-search-trees/

这道其实是dp题。。。左子树的节点数可以是0~n-1个，对应的右子树的节点是n-1~0个。
所以总和s = f(0)*f(n-1) + f(1)*f(n-2) + ... + f(n-2)*f(0)；

换成递推式就是dp[i] = dp[j-1]*dp[i-j] (1<=i<=n, 1<=j<=i) 。
注意basecase dp[0] = 1。

```C++
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n+1);
        dp[0] = 1;

        for(int i=1; i<=n; i++){
            for(int j=1; j<=i; j++){
                dp[i] += dp[j-1] * dp[i-j];
            }
        }
        return dp[n];
        
    }
};
```

## 剑指 Offer 36. 二叉搜索树与双向链表
https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/

维护一个头节点和一个尾节点，然后中序遍历，尾插法把当前节点串成双向链表。

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
    Node* head;
    Node* tail;
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
            head = new Node(root->val);
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

## 109. 有序链表转换二叉搜索树
https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/

有序链表通过快慢指针找到中点，然后按照中点分左右子树，然后按照左右子树进行buildTree操作。

需要注意的是，在链表操作，要把左子树的next置为nullptr。

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

        ListNode* slow = head;
        ListNode* fast = head;
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

## 99. 恢复二叉搜索树
https://leetcode-cn.com/problems/recover-binary-search-tree/

中序遍历，维护一个prev节点，看当前节点和prev节点的大小关系；

如果违反了二叉搜索树的限制，就证明这两个节点是错误的节点。用两个全局变量记住他们。
然后交换他们的值就可以了。


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
    TreeNode* preNode = nullptr;
    TreeNode* s;
    TreeNode *t;
    void recoverTree(TreeNode* root) {
        if(root == nullptr){
            return ;
        }
        traverse(root);
        int temp = s->val;
        s->val = t->val;
        t->val = temp;

    }

    void traverse(TreeNode* root){
        if(root == nullptr){
            return ;
        }
        traverse(root->left);
        if(preNode != nullptr && root->val < preNode->val){
            
            if(s == nullptr){
                s = preNode;
            }
            t = root;
        }
        preNode = root;
        traverse(root->right);
    }
};
```

## 173. 二叉搜索树迭代器
https://leetcode-cn.com/problems/binary-search-tree-iterator/

先将左子树全部入栈；然后每次next的时候，从栈顶取一个出来作为next的值；然后再看它是否有右子树，
如果有，把它的右子树也入栈，再将当前节点改为右子树的左子树。

没有下一个节点的时候就是栈为空的时候。

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
class BSTIterator {
public:
    stack<TreeNode*> subTree;
    BSTIterator(TreeNode* root) {
        while(root != NULL){
            subTree.push(root);
            root = root->left;
        }
    }
    
    /** @return the next smallest number */
    int next() {
        TreeNode* t = subTree.top();
        subTree.pop();
        int val = t->val;
        t = t->right;
        while(t){
            subTree.push(t);
            t = t->left;
        }
        return val;
    }
    
    /** @return whether we have a next smallest number */
    bool hasNext() {
        return !subTree.empty();
    }
};

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator* obj = new BSTIterator(root);
 * int param_1 = obj->next();
 * bool param_2 = obj->hasNext();
 */
 ```

 ## 剑指 Offer 54. 二叉搜索树的第k大节点
 https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/

维护一个全局变量表示是第几个节点，然后按中序遍历遍历就好。遍历到第k大就可以了。

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
    int ans = 0;
    int t;
    int kthLargest(TreeNode* root, int k){
        t = k;
        dfs(root);
        return ans;
    }

    void dfs(TreeNode* root){
        if(root == NULL){
            return ;
        }
        
        dfs(root->right);
        t--;
        if(t == 0){
            ans = root->val;
            return ;
        }
        dfs(root->left);

    }
};
```

## 501. 二叉搜索树中的众数
https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/

中序遍历，如果前一个节点和当前节点一样，那么当前节点的次数加1。

如果当前节点的次数比最大出现次数要大，更新众数和最大次数；同时删除返回的众数；
如果次数相等，则将当前节点也作为众数返回。

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
    int previousVal = 0;
    int currentFreq = 0;
    int maxFreq = 0;
    vector<int> findMode(TreeNode* root) {
        dfs(root);
        return ans;
    }

    void dfs(TreeNode* root){
        if(root == nullptr){
            return ;
        }

        dfs(root->left);
        if(root->val == previousVal){
            currentFreq++;
        }else{
            currentFreq = 1;
            previousVal = root->val;
        }

        if(currentFreq >= maxFreq){
            if(currentFreq > maxFreq){
                ans.clear();
            }
            ans.push_back(root->val);
            maxFreq = currentFreq;
        }

        dfs(root->right);
    }


};
```

## 938. 二叉搜索树的范围和
https://leetcode-cn.com/problems/range-sum-of-bst/

中序遍历，如果当前值比low要小，就往右子树去递归寻找；否则往左子树递归寻找。
如果是在区间内，就往上返回节点的值和左右子树的范围和。

```c++
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

    int ans = 0;
    int rangeSumBST(TreeNode* root, int low, int high) {
        if(root == nullptr){
            return 0;
        }

        if(root->val < low){
            return rangeSumBST(root->right,low,high);
        }
        if(root->val > high){
            return rangeSumBST(root->left,low,high);
        }
        return root->val + rangeSumBST(root->left,low,high) + rangeSumBST(root->right,low,high);
    }
};
```

## 450. 删除二叉搜索树中的节点
https://leetcode-cn.com/problems/delete-node-in-a-bst/

比较大小，从左右子树找到要删除的那个节点。

删除完以后，要将该节点找到最接近它但是比他小的节点。将原来的右子树接到这个节点上。

所以从左子树的右子树开始找，尽可能往右子树开始找。找到以后将原来的右子树接到现在的右子树上返回。

如果没有左子树，直接取右子树就可以。

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
    TreeNode* deleteNode(TreeNode* root, int key) {
        if(root == NULL){
            return NULL;
        }
        if(root->val > key){
            root->left = deleteNode(root->left,key);
        }
        else if(root->val < key){
            root->right = deleteNode(root->right,key);
        }
        else{
            if(root->left){
                TreeNode* node = root->left;
                while(node->right){
                    node = node->right;
                }
                node->right = root->right;
                return root->left;
            }
            return root->right;
        }
        return root;
    }
};
```

## 783. 二叉搜索树节点最小距离
https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/

二叉搜索树的最小距离肯定是中序遍历的两个节点之间产生，所以用一个prev保存前一个节点，每次遍历的时候将当前节点和prev节点的值比较一下就好。

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
    int ans = INT_MAX;
    TreeNode* prev = nullptr;
    int minDiffInBST(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        dfs(root);
        return ans;
    }


    void dfs(TreeNode* root){
        if(root == nullptr){
            return ;
        }

        dfs(root->left);
        if(prev){
            ans = min(ans,root->val - prev->val);
        }
        prev = root;
        dfs(root->right);
    }
};
```

## 538. 把二叉搜索树转换为累加树
https://leetcode-cn.com/problems/convert-bst-to-greater-tree/

累加树要从右边开始累加，所以中序遍历时要从右边开始。保存一个全局变量作为当前总和，每次都将总和和当前节点数值作为新的节点数值。

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
    int sum = 0;
    TreeNode* convertBST(TreeNode* root) {

        if(root != NULL){
            //遍历右子树 
            auto right = convertBST(root->right);
            //将右子树和加到root，因为右子树都是比root大的
            root->val += sum;
            //更新sum，让左子树加。
            sum = root->val;
            //遍历左子树
            auto left = convertBST(root->left);
            return root;

            
        }
        return NULL;
    }

    
};
```

## 230. 二叉搜索树中第K小的元素
https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/

中序遍历，每遍历一个让k-1，当k为0的时候就是第k大的元素；

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
    int ans = 0;
    int kthSmallest(TreeNode* root, int k) {
        if(root == nullptr){
            return 0;
        }
        dfs(root,k);
        return ans;
    }

    void dfs(TreeNode* root, int& k){
        if(root == nullptr || k < 0){
            return ;
        }
        dfs(root->left,k);
        if(--k == 0){
            ans = root->val;
        }
        dfs(root->right,k);
    }
};
```

## 700. 二叉搜索树中的搜索
https://leetcode-cn.com/problems/search-in-a-binary-search-tree/

大的往右找，小的往左找。

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
    TreeNode* searchBST(TreeNode* root, int val) {
        if(root == nullptr){
            return nullptr;
        }
        
        if(root->val < val){
            return searchBST(root->right,val);
        }else if(root->val > val){
            return searchBST(root->left,val);
        }
        return root;
    }
};
```

## 701. 二叉搜索树中的插入操作
https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/

大的往右边插，小的往左边插。

如果插入的地方是空，证明该从这里插进去了，所以新建node节点。

我们返回上一层的是新建的这个节点，所以我们要让它成为root的左子树或者右子树。

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
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root == nullptr){
            auto node = new TreeNode(val);
            return node;
        }

        if(val > root->val){
            root->right = insertIntoBST(root->right,val);
        }
        if(val < root->val){
            root->left = insertIntoBST(root->left,val);
        }
        return root;
    }
};
```

## 669. 修剪二叉搜索树
https://leetcode-cn.com/problems/trim-a-binary-search-tree/

修剪二叉搜索树，如果比low要小，就往右子树修剪；比high大，就往左子树修剪；

如果刚好在这个范围里面，那就对左右子树进行递归调用修剪。

递归函数作用就是修剪这个子树，返回修剪后的root；

所以你的左右子树已经被递归函数的返回值赋值。

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
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(root == nullptr){
            return nullptr;
        }

        if(root->val < low){
            return trimBST(root->right,low,high);
        }
        if(root->val > high){
            return trimBST(root->left,low,high);
        }

        root->left = trimBST(root->left,low,high);
        root->right = trimBST(root->right,low,high);
        return root;
    }
};
```