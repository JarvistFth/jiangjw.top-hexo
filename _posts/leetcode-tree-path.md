---
title: 【树】 - 路径
date: 2020-06-25 17:21:51
categories: 
- leetcode
- 树
tags: [leetcode ,树]
---

树的路径问题。
<!---more--->

## 剑指 Offer 34. 二叉树中和为某一值的路径
https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/

回溯法。每次选取当前节点的值，加入到和里面来。然后看和是不是target，并且是不是叶子结点；

如果是，就将当前路径加入到输出中；否则的话继续往下遍历。

遍历完毕后需要撤销选择，也就是将当前路径减少当前节点。

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
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        if(root == nullptr){
            return {};
        }
        dfs(root,target,0);
        return ans;
    }


    void dfs(TreeNode* root, int target, int sum){
        if(root == nullptr){
            return ;
        }

        sum += root->val;
        path.push_back(root->val);
        if(sum == target && root->left == nullptr && root->right == nullptr){
            ans.push_back(path);
            path.pop_back(); 
            return ;
        }

        dfs(root->left,target,sum);
        dfs(root->right,target,sum);
        path.pop_back();

    }
};
```

## 113. 路径总和 II
https://leetcode-cn.com/problems/path-sum-ii/

题是一样的。

```C++
class Solution {
public:
    vector<int> path;
    vector<vector<int>> ans;
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        if(root == nullptr){
            return {};
        }
        dfs(root,target,0);
        return ans;
    }


    void dfs(TreeNode* root, int target, int sum){
        if(root == nullptr){
            return ;
        }

        sum += root->val;
        path.push_back(root->val);
        if(sum == target && root->left == nullptr && root->right == nullptr){
            ans.push_back(path);
            path.pop_back(); 
            return ;
        }

        dfs(root->left,target,sum);
        dfs(root->right,target,sum);
        path.pop_back();

    }
};
```

## 129. 求根节点到叶节点数字之和
https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/

每次将传入的sum*10 再加上当前节点的值，然后返回这个和给上一层。

这样每层都能获得下一层的所有节点之和。

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
    int sumNumbers(TreeNode* root) {
        if(root == NULL){
            return 0;
        }

        return dfs(root,0);

    }

    int dfs(TreeNode* root,int sum){
        if(root == NULL){
            return 0;
        }
        sum *= 10;
        sum += root->val;
        if(root->left == NULL && root->right == NULL){
            return sum;
        }
        return dfs(root->left,sum) + dfs(root->right,sum);
    }    
};
```

## 257. 二叉树的所有路径
https://leetcode-cn.com/problems/binary-tree-paths/

同样是回溯，先序遍历将当前节点添加到路径里面，然后一直到达叶子结点的时候，将路径输出到ans中。

为了方便处理，这里path就不用全局变量了。

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
    vector<string> ans;
    vector<string> binaryTreePaths(TreeNode* root) {
        dfs(root,"");
        return ans;
    }

    void dfs(TreeNode* root, string path){
        if(root == nullptr){
            return ;
        }
        if(root->left == nullptr && root->right == nullptr){
            path += to_string(root->val);
            ans.push_back(path);
            return ;
        }

        path += to_string(root->val) + "->";

        dfs(root->left,path);
        dfs(root->right,path);
    }
};
```

## 剑指 Offer 33. 二叉搜索树的后序遍历序列
https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/

二叉搜索树的后序序列为：[左子树，右子树，root]； 所以每次我们首先要找到root，然后再找到右子树起始的index。

然后从右子树开始看它是不是都大于root；

然后按照左子树的范围和右子树的范围递归执行就好。

普通解法：
```C++
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        if(postorder.size() < 2){
            return true;
        }
        return dfs(postorder,0,postorder.size()-1);
        

    }

    bool dfs(vector<int>& postorder, int l, int r){
        if(l >= r){
            return true;
        }

        int rootval = postorder[r];
        int idx = l;
        while(idx <= r && postorder[idx] < rootval){
            idx++;
        }
        //[l,idx-1] | [idx .. r]

        for(int i=idx; i<r; i++){
            if(postorder[i] < rootval){
                return false;
            }
        }

        if(!dfs(postorder,l,idx-1)){
            return false;
        }
        if(!dfs(postorder,idx,r-1)){
            return false;
        }
        
        return true;

    }
        
};
```

单调栈解法类似。

单调栈解法：
```C++
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        if(postorder.size() < 2){
            return true;
        }

        stack<int> s;
        int prev = INT_MAX;

        for(int i=postorder.size()-1 ; i>=0 ;i--){
            int rootval = postorder[i];
            if(rootval > prev){
                return false;
            }

            while(!s.empty() && rootval < s.top()){
                prev = s.top();
                s.pop();
            }
            s.push(rootval);
        }
        return true;

    }
        
};
```

## 897. 递增顺序搜索树
https://leetcode-cn.com/problems/increasing-order-search-tree/

新建一个头节点，然后按照中序遍历，每次将它尾插法到后面。

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
    TreeNode* dummy = new TreeNode(-1);
    TreeNode* curr = dummy;
    TreeNode* increasingBST(TreeNode* root) {
        if(root == nullptr){
            return root;
        }

        increasingBST(root->left);
        
        curr->right = new TreeNode(root->val);
        curr->left = nullptr;
        curr = curr->right;
        increasingBST(root->right);
        return dummy->right;
    }
};
```

## 437. 路径总和 III
https://leetcode-cn.com/problems/path-sum-iii/

和路径总和II是一样的。只是需要双重递归，对左右子树都pathsum一下。

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
    int pathSum(TreeNode* root, int targetSum) {
        if(root == nullptr){
            return 0;
        }
        dfs(root,targetSum);
        pathSum(root->left,targetSum);
        pathSum(root->right,targetSum);
        return ans;
    }


    void dfs(TreeNode* root, int targetSum){
        if(root == nullptr){
            return ;
        }
        targetSum -= root->val;
        if(targetSum == 0){
            ans++;
        }
        dfs(root->left,targetSum);
        dfs(root->right,targetSum);
    }
};


//优化解法：
//用一个map存储和为某一值的路径条数。遍历的时候计算当前路径的总和；
//然后看当前和 - 目标和 的差 是否在map中存在；
//如果存在，证明之前是有一条和为这个差的路径存在。【目标和 + ？ == 当前和】，如果这个？存在，那么目标和也一定存在。对应路径条数就是map的value。

//注意回溯的时候要减回路径条数。
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
    unordered_map<int,int> prefix;

    int pathSum(TreeNode* root, int targetSum) {
        if(root == nullptr){
            return 0;
        }
        prefix[0]=1;
        dfs(root,targetSum,0);
        return ans;
    }

    void dfs(TreeNode* root, int targetSum, int currentSum){
        if(root == nullptr){
            return ;
        }
        currentSum += root->val;
        if(prefix.count(currentSum - targetSum)){
            ans += prefix[currentSum - targetSum];
        }
        prefix[currentSum]++;
        dfs(root->left,targetSum,currentSum);
        dfs(root->right,targetSum,currentSum);
        prefix[currentSum]--;
    }
};

```

## 687. 最长同值路径
https://leetcode-cn.com/problems/longest-univalue-path/

后续遍历像找高度那样。但是注意只有当前节点的值和前一个节点的值相等的时候，才返回左右子树的高度+1；如果不相等，证明不是同值路径，返回0.

递归函数返回的就是左右子树的最长同值路径长度，

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
    int longestUnivaluePath(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        dfs(root,root->val);
        return ans;
    }

    int dfs(TreeNode* root, int rootval){
        if(root == nullptr){
            return 0;
        }
        int left = dfs(root->left,root->val);
        int right = dfs(root->right, root->val);
        ans = max(left+right,ans);
        return rootval==root->val?max(left,right)+1:0;

    }
};
```