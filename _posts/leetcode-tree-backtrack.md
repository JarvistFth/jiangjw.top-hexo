---
title: leetcode-tree-backtrack
date: 2020-07-13 00:25:39
categories: 
- leetcode
- 树
tags: [leetcode,树]
---
遍历过程可能需要回溯，核心是，遍历前做选择；遍历后取消选择。
<!---more--->


### 1457. 二叉树中的伪回文路径
https://leetcode-cn.com/problems/pseudo-palindromic-paths-in-a-binary-tree/

用map或者数组保存遍历过的路径的节点值，因为题目要求是可以调换节点顺序，所以不用严格回文，只需要判断一下保存下来的节点的值只有一个是奇数的就可以了。

注意的是在遍历前做选择，对root->val保存在map或者数组中的值++；在遍历后要取消选择（--）。

```C++
class Solution {
public:
    int ans = 0;
    map<int,int> vals;
    int pseudoPalindromicPaths (TreeNode* root) {
        dfs(root);
        return ans;
    }

    void dfs(TreeNode* root){
        if(root == nullptr){
            return;
        }
        vals[root->val]++;
        if(root->left == nullptr && root->right == nullptr){
            ans += ispseudoPalindromic();
        }
        dfs(root->left);
        dfs(root->right);
        vals[root->val]--;


    }

    bool ispseudoPalindromic(){
        bool two_odd = false;
        for(auto v:vals){
            if(v.second % 2 == 0){
                continue;
            }
            if(two_odd){
                return false;
            }
            two_odd = true;
        }
        return true;
    }
};
```