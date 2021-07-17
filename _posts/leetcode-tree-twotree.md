---
title: 【树】 - 镜像/翻转/
date: 2020-06-25 17:19:31
categories: 
- leetcode
- 树
tags: [leetcode,树]
---
树的镜像/翻转/子结构 等特性，涉及两棵树进行比较的类型的题目。
<!---more--->

## 226. 翻转二叉树
https://leetcode-cn.com/problems/invert-binary-tree/

每个节点交换左右子树就可以了。

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
    TreeNode* invertTree(TreeNode* root) {
        if(root == NULL){
            return NULL;
        }
        
        // auto right = root->right;
        // root->right = invertTree(root->left);
        // root->left = invertTree(right);
        swap(root->left,root->right);
        invertTree(root->left);
        invertTree(root->right);
        return root;
    }
};
```

## 101. 对称二叉树
https://leetcode-cn.com/problems/symmetric-tree/

递归比较，分别比较当前节点的左子树和当前节点翻转后的左子树（也就是右子树）。

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

## 剑指 Offer 26. 树的子结构
https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/

双重递归，第一层递归用于遍历每个节点，对每个节点比较当前的树结构和给定树结构是否一致。

另一个递归就是用作内部比较两个树的结构是否一致。

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
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if(A == NULL || B == NULL){
            return false;
        }
        return dfs(A,B) || isSubStructure(A->left,B) || isSubStructure(A->right,B);
    }

    bool dfs(TreeNode* A,TreeNode* B){
        if(B == NULL){
            return true;
        }
        if(A == NULL){
            return false;
        }
        if(A->val != B->val){
            return false;
        }
        return dfs(A->left,B->left) && dfs(A->right,B->right);
    }
};
```

## 剑指 Offer 27. 二叉树的镜像
https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/

翻转和镜像是一样的。
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
    TreeNode* mirrorTree(TreeNode* root) {
        if(root == NULL){
            return NULL;
        }
        // auto left = root->left;
        // root->left = mirrorTree(root->right);
        // root->right = mirrorTree(left);
        // return root;
        swap(root->left,root->right);
        mirrorTree(root->left);
        mirrorTree(root->right);
        return root;
    }

    
};
```

## 617. 合并二叉树
https://leetcode-cn.com/problems/merge-two-binary-trees/

先序遍历，比较两棵的节点，将他们的和加起来，然后返回给上一层调用。

这样当前的左子树和右子树就是递归的返回值。

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
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if(!t1 && !t2){
            return NULL;
        }

        if(!t1){
            return t2;
        }
        if(!t2){
            return t1;
        }

        t1->val += t2->val;
        t1->left = mergeTrees(t1->left,t2->left);
        t1->right = mergeTrees(t1->right,t2->right);
        return t1;
    }

    
};
```

## 100. 相同的树
https://leetcode-cn.com/problems/same-tree/

普通先序遍历，看每个节点是不是一样的。

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
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if(p == NULL && q == NULL){
            return true;
        }
        if(p == NULL || q == NULL){
            return false;
        }
        if(p->val != q->val){
            return false;
        }

        return isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
    }
};
```

## 872. 叶子相似的树
https://leetcode-cn.com/problems/leaf-similar-trees/

用两个vector存一下两棵树的叶子结点，然后比较一下。

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
    vector<int> v1,v2;
    bool leafSimilar(TreeNode* root1, TreeNode* root2) {
        dfs(root1,v1);
        dfs(root2,v2);
        return v1==v2;
    }


    void dfs(TreeNode* root,vector<int>& vec){
        if(!root){
            return ;
        }
        if(!root->left && !root->right){
            vec.push_back(root->val);
            return ;
        }
        dfs(root->left,vec);
        dfs(root->right,vec);

        
    }
};
```
## 331. 验证二叉树的前序序列化
https://leetcode-cn.com/problems/verify-preorder-serialization-of-a-binary-tree/

根结点的入度为0出度为2，其他非叶子结点的入度为1出度为2，叶子节点入度为1出度为0。因为根节点多出来一个出度，所以初始化度为1，一个非叶子节点时度+1，加入一个空节点（叶子节点）时度-1，如果度为0，即达到出度入度相等，已经形成一颗二叉树。

```C++
class Solution {
public:
    bool isValidSerialization(string preorder) {
        stringstream sstr(preorder);
        string str;

        int degree = 1;

        while (getline(sstr, str, ',')) {
            if(degree == 0){
                return false;
            }
            if(str == "#"){
                degree--;
            }
            else{
                degree++;
            }
        }
        return degree == 0;
    }
};
```

## 572. 另一个树的子树
https://leetcode-cn.com/problems/subtree-of-another-tree/

双重递归，主递归负责递归比较每个节点是否有可能含有子树；副递归负责比较两棵树的值是否相等以及节点结构是否一致。

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
    bool isSubtree(TreeNode* root, TreeNode* subRoot) {
        if(root == nullptr || subRoot == nullptr){
            return false;
        }
        return dfs(root,subRoot) || isSubtree(root->left,subRoot) || isSubtree(root->right,subRoot);
    }

    bool dfs(TreeNode* s, TreeNode* t){
        if(!s && !t){
            return true;
        }
        else if(!s || !t){
            return false;
        }
        if(s->val != t->val){
            return false;
        }
        return dfs(s->left,t->left) && dfs(s->right,t->right);
    }
};
```

## 652. 寻找重复的子树
https://leetcode-cn.com/problems/find-duplicate-subtrees/

后序遍历，将所有子树用字符串做key，用哈希表保存；如果哈希表中有这个字符串，证明有重复的子树。

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
    vector<TreeNode*> ans;
    unordered_map<string,int> m;
    vector<TreeNode*> findDuplicateSubtrees(TreeNode* root) {
        if(root == nullptr){
            return ans;
        }
        dfs(root);
        return ans;
    }

    string dfs(TreeNode* root){
        if(root == nullptr){
            return "";
        }
        string rootstring = to_string(root->val) + ' ' + dfs(root->left) + ' ' + dfs(root->right);
        if(m[rootstring] == 1){
            ans.push_back(root);
        }
        m[rootstring]++;
        return rootstring;
        

    }
};
```

## 951. 翻转等价二叉树
https://leetcode-cn.com/problems/flip-equivalent-binary-trees/

就是传入左子树-左子树&&右子树-右子树 || 左子树-右子树&&右子树-左子树；看他们是不是相等。。

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
    bool flipEquiv(TreeNode* root1, TreeNode* root2) {
        if(root1 == NULL && root2 == NULL){
            return true;
        }
        if(root1 == NULL || root2 == NULL || root1->val != root2->val){
            return false;
        }
        return (flipEquiv(root1->left,root2->left) && flipEquiv(root1->right,root2->right))
            || (flipEquiv(root1->left,root2->right) && flipEquiv(root1->right,root2->left));
    }

    
};
