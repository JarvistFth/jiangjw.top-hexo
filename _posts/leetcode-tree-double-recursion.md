---
title: leetcode-tree-double-recursion
date: 2020-06-28 23:31:55
categories: leetcode-tree
tags: [leetcode ,树]
---
一些问题经常会用到双重递归的办法，一般都是要每个根节点都与一个列表/树进行比较，外层递归负责移动root，里层递归负责对比。
<!---more--->
## 1367. 二叉树中的列表
https://leetcode-cn.com/problems/linked-list-in-binary-tree/

拿到手第一个想法是一趟递归完成，后来发现isSubPath()只操作左右子树会比较好，如果还要操作head进行比对，写起来会很复杂；最好是用另一个递归函数去处理列表的head。
```C++

class Solution {
public:
    //外层递归，负责移动root
    bool isSubPath(ListNode* head, TreeNode* root) {
        if(head == NULL){
            return true;
        }
        if(root == NULL){
            return false;
        }
        return dfs(head,root) || 
        isSubPath(head,root->left) || 
        isSubPath(head,root->right);
    }
    //里层递归，负责移动list->head
    bool dfs(ListNode* head, TreeNode* root){
        if(head == NULL){
            return true;
        }
        if(root == NULL){
            return false;
        }
        if(head->val != root->val){
            return false;
        }
        return dfs(head->next,root->left) || dfs(head->next,root->right);
    }
};
```
## 剑指 Offer 26. 树的子结构
https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/

给出的函数是判断从root判断是否包含子树，所以肯定是递归调用左子树&&递归调用右子树；

对每一次递归调用时，都需要再次从这次的root递归遍历子树，与给出的B子树每个子树节点进行对比；所以是双重递归

```C++

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