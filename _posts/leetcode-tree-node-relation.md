---
title: leetcode-tree-node-relation
date: 2020-06-25 17:21:33
categories: 
- leetcode
- 树
tags: [leetcode ,树]
    
---
树的节点关系系列问题
<!---more--->

### 面试题 04.08. 首个共同祖先
https://leetcode-cn.com/problems/first-common-ancestor-lcci/

给出的函数是
```C++
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q){

}
```
返回值是p和q的首个公共祖先；

什么时候返回：root为null时，返回null；如果root==p||root==q 意味找到了p q子节点，返回当前节点-root；

然后递归调用左右子树。如果左右都有返回值，证明在左右都分别找到了p和q，返回root；如果左右有一个没找到，证明p和q的公共祖先在找到那一侧子树中，返回对应的left或者right。

```C++

class Solution {
public:
    TreeNode* ans;
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == NULL){
            return root;
        }
        if(root == p || root == q){
            return root;
        }
        TreeNode* left = lowestCommonAncestor(root->left,p,q);
        TreeNode* right = lowestCommonAncestor(root->right,p,q);
        
        
        //左右找到了pq
        if(left != NULL && right != NULL){
            return root;
        }
        //右边找不到
        if(left != NULL && right == NULL){
            return left;
        }
        return right;
        
    }
};
```

### 993. 二叉树的堂兄弟节点
https://leetcode-cn.com/problems/cousins-in-binary-tree/

遍历节点，找到给定值的节点时，记录其父节点和高度；
最后比较父节点和高度看是不是堂兄弟节点。

```C++

class Solution {
public:
    int hx = -1, hy = -1;
    TreeNode* px = nullptr;
    TreeNode* py = nullptr;
    bool isCousins(TreeNode* root, int x, int y) {
        dfs(root,x,y,0);
        return (hx == hy) && (px != py);
    }

    void dfs(TreeNode* root ,int x,int y,int h){
        if(root == nullptr){
            return;
        }
        if(root->left && root->left->val == x){
            px = root;
            hx = h;
        }
        if(root->right && root->right->val == y){
            py = root;
            hy = h;
        }
        if(root->left && root->left->val == y){
            py = root;
            hy = h;
        }
        if(root->right && root->right->val == x){
            px = root;
            hx = h;
        }
        dfs(root->left,x,y,h+1);
        dfs(root->right,x,y,h+1);
    }
};
```