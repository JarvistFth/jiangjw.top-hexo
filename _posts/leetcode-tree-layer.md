---
title: 【树】- 层序遍历相关
date: 2020-06-25 13:53:07
categories: 
- leetcode
- 树
tags: [leetcode ,树]
keywords: [leetcode,树]

---

树的层序遍历相关相关题目。
<!---more--->

## 199. 二叉树的右视图 
https://leetcode-cn.com/problems/binary-tree-right-side-view/

层序遍历，每一层的坐标是最后一个的时候将它push_back()。

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
    queue<TreeNode*> levelQueue;
    vector<int> ans;
    vector<int> rightSideView(TreeNode* root) {
        if(root == NULL){
            return ans;
        }
        levelQueue.push(root);
        while(!levelQueue.empty()){
            int size = levelQueue.size();
            for(int i=0;i<size;i++){
                TreeNode* node = levelQueue.front();
                levelQueue.pop();
                if(node->left){
                    levelQueue.push(node->left);
                }
                if(node->right){
                    levelQueue.push(node->right);
                }
                if(i == size - 1){
                    ans.push_back(node->val);
                }
            }
        }

        return ans;
    }
};
```

## 103. 二叉树的锯齿形层序遍历
https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/

层序遍历，维护一个当前的高度，如果是奇数，就reverse当前那一层。

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
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if(root == NULL){
            return {};
        }
        queue<TreeNode*> q;

        q.push(root);
        vector<vector<int>> ans;
        int k = 0;
        while(!q.empty()){
            
            int n = q.size();
            vector<int> layer;
            for(int i=0;i<n;i++){
                auto front = q.front();
                q.pop();
                layer.push_back(front->val);
                if(front->left){
                    q.push(front->left);
                }
                if(front->right){
                    q.push(front->right);
                }
                
            }
            if(k++ % 2 != 0){
                reverse(layer.begin(),layer.end());
            }
            ans.push_back(layer);
        }
        return ans;
    }
};
```

## 剑指 Offer 32 - I. 从上到下打印二叉树 
https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/

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
    vector<int> levelOrder(TreeNode* root) {
        if(root == NULL){
            return {};
        }
        vector<int> ans;
        queue<TreeNode*> levelQueue;
        levelQueue.push(root);

        while(!levelQueue.empty()){
            int size = levelQueue.size();
            for(int i=0; i<size; i++){
                auto front = levelQueue.front();
                levelQueue.pop();
                ans.push_back(front->val);
                if(front->left){
                    levelQueue.push(front->left);
                }
                if(front->right){
                    levelQueue.push(front->right);
                }
            }
        }
        return ans;
    }
};
```

## 958. 二叉树的完全性检验
https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/

将每一层都加入到队列中进行层序遍历。当第一次出现nullptr的时候，判断一下队列里面还有没有没有遍历到的节点，如果有就认为是false；否则是true。

这里加一个flag表示第一次碰到nullptr的时候，就不要将它的左右子树加进去。如果后面还有nullptr，证明二叉树中间是空的，return false。

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
    bool isCompleteTree(TreeNode* root) {
        queue<TreeNode*> q;

        q.push(root);
        bool flag = false;
        while(!q.empty()){
            auto front = q.front();
            q.pop();
            if(front == nullptr){
                flag = true;
                continue;
            }
            if(flag){
                return false;
            }
            q.push(front->left);
            q.push(front->right);
        }
        return true;
    }
};
```

## 面试题 04.03. 特定深度节点链表
https://leetcode-cn.com/problems/list-of-depth-lcci/

层序遍历，每一层建一个链表放入vector中。

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
    vector<ListNode*> ans;
    queue<TreeNode*> q;
    vector<ListNode*> listOfDepth(TreeNode* tree) {
        if(tree == NULL){
            return {};
        }
        ListNode* head = new ListNode(tree->val);
        q.push(tree);
        ans.push_back(head);
        while(!q.empty()){
            ListNode* begin = new ListNode(-1);
            ListNode* current = begin;
            auto len = q.size();
            for(int i=0;i<len;i++){
                auto t = q.front();
                q.pop();
                if(t->left != NULL){
                    current -> next = new ListNode(t->left->val);
                    current = current ->next;
                    q.push(t->left);
                }
                if(t->right != NULL){
                    current -> next = new ListNode(t->right->val);
                    current = current ->next;
                    q.push(t->right);
                }
            }
            if(begin->next){
                ans.push_back(begin->next);
            }
        }
        return ans;
    }
};
```