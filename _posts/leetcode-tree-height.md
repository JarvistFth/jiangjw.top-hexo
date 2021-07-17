---
title: 【树】 - 深度/高度/宽度
date: 2020-06-23 13:55:54
categories: 
- leetcode
- 树
tags: [leetcode ,树]

---
树的深度、高度、宽度相关题目。
<!---more--->

## 104. 二叉树的最大深度 || 剑指 Offer 55 - I. 二叉树的深度
https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/

https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/

每遍历一次向上一层调用返回+1作为深度+1 。

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

## 110. 平衡二叉树
https://leetcode-cn.com/problems/balanced-binary-tree/

递归法，后序遍历只用O(n)时间复杂度；每次获取左右子树高度，看绝对值是否在1以内。

面试还被问道记忆化搜索的先序遍历，具体来说就是为每个Node建立一个备忘录，记录它当前的高度。这样就可以避免重复计算。。

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
    bool ans = true;
    bool isBalanced(TreeNode* root) {
        if(root == NULL){
            return true;
        }
        getDepth(root);
        return ans;
    }

    int getDepth(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int left = getDepth(root->left);
        int right = getDepth(root->right);
        if(abs(left - right) > 1){
            ans = false;
        }
        return left>right?left+1:right+1;
    }
};
```

##  662. 二叉树最大宽度
https://leetcode-cn.com/problems/maximum-width-of-binary-tree/

这题宽度需要计算空结点，所以在遍历每一层的时候，将高度传入；维护一个vector记录该层最左边的坐标，和最右边的坐标，然后取他们之间的差+1作为宽度。再维护一个最大宽度进行比较。

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
    typedef unsigned long long ull;
    // ull maxWidth = 1;
    // queue<pair<TreeNode*,ull>> levelQueue;
    vector<ull> vec;
    ull maxWidth = 1;
    int widthOfBinaryTree(TreeNode* root) {
    //     if(root == nullptr){
    //         return 0;
    //     }
    //     levelQueue.push({root,1});
    //     while(!levelQueue.empty()){
    //         int size = levelQueue.size();
    //         if(size == 1){
    //             auto nodeP = levelQueue.front();
    //             levelQueue.pop();
    //             levelQueue.push({nodeP.first,1});
    //         }
    //         ull leftidx,rightidx;
    //         for(int i=0;i<size;i++){
    //             auto nodeP = levelQueue.front();
    //             levelQueue.pop();
    //             auto node = nodeP.first;
    //             if(i == 0){
    //                 leftidx = nodeP.second;
    //             }
    //             if(i == size-1){
    //                 rightidx = nodeP.second;
    //             }

    //             if(node->left){
    //                 levelQueue.push({node->left,(2*nodeP.second)});
    //             }
    //             if(node->right){
    //                 levelQueue.push({node->right,(2*nodeP.second+1)});
    //             }

    //         }
    //         maxWidth = max(maxWidth,(rightidx - leftidx + 1));
    //     }
    //     return maxWidth;

        dfs(root,1,1);
        return maxWidth;


    }

    void dfs(TreeNode* root,ull level, ull index){
        if(root == nullptr){
            return;
        }
        if(level > vec.size()){
            vec.push_back(index);
        }
        maxWidth = max(maxWidth,index - vec[level-1]+1);
        dfs(root->left,level+1,2*index);
        dfs(root->right,level+1,2*index+1);
    }
};
```

## 111. 二叉树的最小深度
https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/

和求最大高度类似，但是要注意，最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
如果左右子树不存在，是不符合要求的。所以要增加条件，判断其左右子树是否存在。

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
    int minDepth(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        if(root->left == NULL && root->right == NULL){
            return 1;
        }
        if(root->right == NULL && root->left != NULL){
            return 1+minDepth(root->left);
        }
        if(root->left == NULL && root->right != NULL){
                return 1+minDepth(root->right);
        }
        return min(minDepth(root->left),minDepth(root->right))+1;
    }
};
```

## 222. 完全二叉树的节点个数
https://leetcode-cn.com/problems/count-complete-tree-nodes/

完全二叉树的节点个数和完全二叉树的高度有关。如果它这一层是满的，那么节点数就是2^(h-1)个。
所以我们可以通过左右子树的高度来判断，如果左右子树高度相等，证明它的左子树是满的；左子树也不满。

对于左子树是满的情况，我们再去对右子树来计算节点个数；

如果左子树不满，这时候左子树高度比右子树高，我们取右子树作为高度计算这一层节点个数，再递归调用计算左子树的个数。

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
    int countNodes(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        int l = getDepth(root->left);
        int r = getDepth(root->right);
        if(l == r){
            return (1<<l) + countNodes(root->right);
        }
        return (1<<r) + countNodes(root->left);


    }

    int getDepth(TreeNode* root){
        int depth = 0;
        while(root){
            depth++;
            root = root->left;
        }
        return depth;
    }
};
```

## 563. 二叉树的坡度
https://leetcode-cn.com/problems/binary-tree-tilt/

坡度是左右子树的和之差；即ans += abs(left - right)；
所以我们递归返回的是左右子树的和。即root->val+left+right；递归函数的作用就是对左右子树求和

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
    int findTilt(TreeNode* root) {

        dfs(root);
        return ans;
    }

    int dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        auto left = dfs(root->left);
        auto right = dfs(root->right);
        ans += abs(left - right);
        return root->val + left + right;
    }
};
```