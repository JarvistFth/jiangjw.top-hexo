---
title: 【树】 - 直径
date: 2020-07-11 12:42:43
categories: 
- leetcode
- 树
tags: [leetcode,树]
---
树的直径
<!---more--->

## 543. 二叉树的直径
https://leetcode-cn.com/problems/diameter-of-binary-tree/

直径长度 = 左子树高度+右子树高度。所以递归求的是子树高度，存一个ans作为最大直径，每次比较一下左右子树的和就可以了。

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
    int ans = INT_MIN;
    int diameterOfBinaryTree(TreeNode* root) {
        dfs(root);
        return ans;
    }

    int dfs(TreeNode* root){
        if(root == nullptr){
            return 0;
        }

        int left = dfs(root->left);
        int right = dfs(root->right);
        ans = max(ans,left+right);
        return max(left,right)+1;
    }
};
```

