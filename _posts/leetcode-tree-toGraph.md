---
title: leetcode-tree-toGraph
date: 2020-07-11 12:42:43
categories: 
- leetcode
- 树
tags: [leetcode,树]
---


树转图，一般还会加深搜
<!---more--->

### 863. 二叉树中所有距离为 K 的结点
https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/

对树的每个节点建图，用map保存节点的父节点；对每个节点转成图，得到targetNode的位置，往其父节点、左子树、右子树进行dfs。在dfs过程中，如果已经遍历过的节点跳过，所以要用set保存访问过的节点；不断地dfs三个方向，参数不断传入距离K-1

```C++
class Solution {
public:
    map<TreeNode*,TreeNode*> parents;
    set<TreeNode*> used;
    TreeNode* targetNode;

    vector<int> ans;

    vector<int> distanceK(TreeNode* root, TreeNode* target, int K) {
       findParents(root,NULL,target);
       dfs(targetNode,K);
       return ans;
    }

    void findParents(TreeNode* root, TreeNode* parent, TreeNode* target){
        if(root == NULL){
            return;
        }
        if(root->val == target->val){
            targetNode = root;
        }
        parents[root] = parent;
        findParents(root->left,root,target);
        findParents(root->right,root,target);
    }

    void dfs(TreeNode* root,int distance){
        if(root != NULL && used.count(root) == 0){
            used.insert(root);
            if(distance <= 0){
                ans.push_back(root->val);
                return;
            }
            dfs(root->left,distance-1);
            dfs(root->right,distance-1);
            dfs(parents[root],distance-1);
        }
        
    }
};
```