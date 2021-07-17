---
title: 【树】 - 基本操作
date: 2020-06-25 13:53:07
categories: 
- leetcode
- 树
tags: [leetcode ,树]
keywords: [leetcode,树]


---
二叉树基本遍历方法。
<!---more--->

## 递归原则
递归三要素。
1. 这个函数是干什么的：
2. 什么时候返回：
3. 返回值是什么：

## 基本遍历方法：
递归太简单了就不写了，非递归的模板如下：
```C++
TreeNode* node = root;
while(node|| !stack.empty()){
    if(node){
        //这里遍历左子树 先序遍历遍历之前（在这里）就对节点的值处理了

    }else{
        //取栈顶 遍历右子树 中序遍历在这里出栈后处理节点的值
        //后序遍历需要判断是左子树叶子节点或者是根节点再处理
    }
}
```


#### 先序遍历



```C++


class Solution {
public:
    vector<int> ans;
    stack<TreeNode*> nodeStack;
    vector<int> preorderTraversal(TreeNode* root) {
        if(root == NULL){
            return {};
        }
        // ans.push_back(root->val);
        // preorderTraversal(root->left);
        // preorderTraversal(root->right);
        // return ans;

        // nodeStack.push(root);
        auto node = root;
        while(node || !nodeStack.empty()){
            if(node){
                nodeStack.push(node);
                ans.push_back(node->val);
                node = node ->left;
            }
            else{
                node = nodeStack.top();
                nodeStack.pop();
                node = node->right;
            }
        }
        return ans;
    }
};
```

### 中序遍历
```C++

class Solution {
public:
    vector<int> ans;
    stack<TreeNode*> nodeStack;
    vector<int> inorderTraversal(TreeNode* root) {
        if(root == NULL){
            return {};
        }
        auto node = root;
        while(node || !nodeStack.empty()){
            if(node){
                nodeStack.push(node);
                node = node->left;
            }else{
                node = nodeStack.top();
                nodeStack.pop();
                ans.push_back(node->val);
                node = node -> right;
            }
        }
        return ans;
    }
};
```

### 后序遍历
```C++

class Solution {
public:
    vector<int> ans;
    stack<TreeNode*> nodeStack;
    vector<int> postorderTraversal(TreeNode* root) {
        TreeNode* node = root;
        TreeNode* pre = NULL;
        while(node || !nodeStack.empty()){
            if(node){
                nodeStack.push(node);
                node = node->left;
            }
            else{
                node = nodeStack.top();
                //先遍历右子树，如果右子树为空或者右子树已经遍历过
                if(node->right == NULL || node->right == pre){
                    ans.push_back(node->val);
                    nodeStack.pop();
                    //这里注意要将现在的node置空                    
                    pre = node;
                    node = NULL;
                }else{
                    node = node->right;
                }
            }
        }
        return ans;
    }
};
```
### 层序遍历 
https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

```C++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if(root == nullptr){
            return {};
        }

        vector<vector<int>> ans;
        queue<TreeNode*> levelQueue;
        levelQueue.push(root);

        while(!levelQueue.empty()){
            int size = levelQueue.size();
            vector<int> level;

            for(int i=0; i<size;i++){
                auto top = levelQueue.front();
                levelQueue.pop();
                level.push_back(top->val);
                if(top->left){
                    levelQueue.push(top->left);
                }
                if(top->right){
                    levelQueue.push(top->right);
                }
            }
            ans.push_back(level);
        }
        return ans;

    }
};
```

## 589. N 叉树的前序遍历
https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/

前序遍历：先遍历根节点，然后遍历左子树，再遍历右子树；

N叉树就是也是从左到右遍历子树。

因此，非递归遍历是每次将当前结点右孩子节点和左孩子节点依次压入栈中。

```C++
/*
// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> children;

    Node() {}

    Node(int _val) {
        val = _val;
    }

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/

class Solution {
public:
    vector<int> preorder(Node* root) {
        if(root == nullptr){
            return {};
        }

        vector<int> ans;
        stack<Node*> stk;
        stk.push(root);

        while(!stk.empty()){
            auto top = stk.top();
            stk.pop();
            ans.push_back(top->val);
            for(int i=top->children.size()-1; i>=0; i--){
                stk.push(top->children[i]);
            }
        }
        return ans;

    }
};
```