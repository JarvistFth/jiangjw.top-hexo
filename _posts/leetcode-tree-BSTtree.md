---
title: leetcode-tree-BSTtree
date: 2020-06-25 17:22:20
categories: leetcode-tree
tags: 
    leetcode 
    树
---

二叉搜索树
<!---more--->


## 面试题 04.06. 后继者
https://leetcode-cn.com/problems/successor-lcci/

二叉搜索树基本都是利用中序遍历得到排序访问。
```C++
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
        if(root != NULL){
            //左子树中找下一个节点，返回的是找到的节点；找到就返回，找不到证明该节点在root或者有节点；
            auto res = inorderSuccessor(root->left,p);
            if(res != NULL){
                return res;
            }
            //右子树的值都比root大，我们要找的是第一个比p大的，如果root大于p，直接返回root
            if(root->val > p->val){
                return root;
            }
            //一直往右子树查找，找比root大的
            return inorderSuccessor(root->right,p);
        }
        return NULL;

    }

};
```
## 面试题 04.05. 合法二叉搜索树
https://leetcode-cn.com/problems/legal-binary-search-tree-lcci/

中序遍历需要记录前一个节点的值做比较的时候，需要一个全局变量来记录前一个节点的值


```C++

class Solution {
public:
    TreeNode* pre = NULL;
    bool isValidBST(TreeNode* root) {
        if(root != NULL){
            if(!isValidBST(root->left)){
                return false;
            };
            if(pre!=NULL && pre->val >= root->val){
                return false;
            }
            pre = root;
            if(!isValidBST(root->right)){
                return false;
            }
            return true;
        }
        return true;
    }
};
```

## 98. 验证二叉搜索树
https://leetcode-cn.com/problems/validate-binary-search-tree/
和上题一样。

## 530. 二叉搜索树的最小绝对差
https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/

BST，照旧中序遍历；因为是二叉搜索树，所以最小绝对差肯定是在root和其子树之间产生，越往下绝对差越大。所以和上面的类似，用一个节点记录上一次遍历的节点的值，然后和当前遍历的节点的值作差，再设定一个全局最小值用于比较取最小值。

```C++

class Solution {
public:
    int ans = INT_MAX;
    TreeNode* preNode = NULL;
    int getMinimumDifference(TreeNode* root) {
        dfs(root);
        return ans;
    }

    void dfs(TreeNode *root){
        if(root == NULL){
            return ;
        }
        dfs(root->left);
        if(preNode != NULL){
            ans = min(ans,root->val - preNode->val);
        }
        preNode = root;
        dfs(root->right);
    }
};
```
## 538. 把二叉搜索树转换为累加树
https://leetcode-cn.com/problems/convert-bst-to-greater-tree/

这个和普通的中序遍历不太一样，因为他是把所有大于它的节点的值加到当前节点上，所以要先遍历右子树，得到最大的值，然后再遍历根节点和左子树，这样可以依次得到从大到小的序列，就比较好累加。

```C++

class Solution {
public:
    int sum = 0;
    TreeNode* convertBST(TreeNode* root) {

        if(root != NULL){
            //遍历右子树 
            auto right = convertBST(root->right);
            //将右子树和加到root，因为右子树都是比root大的
            root->val += sum;
            //更新sum，让左子树加。
            sum = root->val;
            //遍历左子树
            auto left = convertBST(root->left);
            return root;
        }
        return NULL;
    }
};
```

## 173. 二叉搜索树迭代器
https://leetcode-cn.com/problems/binary-search-tree-iterator/


题目要求空间复杂度为O(h)，所以用数组保存所有中序遍历的节点然后依次返回的方法不合题意！

所以想办法，用数组或者栈保存左子树的节点。因为最小的值，就是一直递归左子树到叶子节点，正好是O(h)的空间。next()时，返回数组的最后一位，将当前节点指针指向右子树（因为可能还存在右子树，这个节点比上一个入栈的左子树要小）；然后重复遍历右子树的左子树。
```C++
class BSTIterator {
public:
    vector<TreeNode*> subTree;
    BSTIterator(TreeNode* root) {
        while(root != NULL){
            subTree.push_back(root);
            root = root->left;
        }
    }
    
    /** @return the next smallest number */
    int next() {
        TreeNode* t = subTree.back();
        subTree.pop_back();
        int val = t->val;
        t = t->right;
        while(t){
            subTree.push_back(t);
            t = t->left;
        }
        return val;
    }
    
    /** @return whether we have a next smallest number */
    bool hasNext() {
        return !subTree.empty();
    }
};
```

## 938. 二叉搜索树的范围和
https://leetcode-cn.com/problems/range-sum-of-bst/

```C++

class Solution {
public:
    int rangeSumBST(TreeNode* root, int L, int R) {
        if(root == NULL){
            return 0;
        }

        //如果根节点的值大于要搜的R，说明要往小的方向搜，所以往左边搜
        if(root->val > R){
            return rangeSumBST(root->left,L,R);
        }
        //同理，如果已经小于L，说明往大了搜
        else if(root->val < L){
            return rangeSumBST(root->right,L,R);
        }
        //如果不大不小，说明在范围内，返回root+左边返回的节点和+右边的节点和
        return root->val + rangeSumBST(root->left,L,R) + rangeSumBST(root->right,L,R);

    }
};
```

## 701. 二叉搜索树中的插入操作
https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/

判断节点大小，然后递归调用，返回root节点，将新建的节点作为root的左右节点。
```C++
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root == nullptr){
            TreeNode* node = new TreeNode(val);
            return node;
        }


        if(val>root->val){
            root->right = insertIntoBST(root->right,val);
        }
        if(val<root->val){
            root->left = insertIntoBST(root->left,val);
        }
        return root;
    }
};
```


## 1305. 两棵二叉搜索树中的所有元素
https://leetcode-cn.com/problems/all-elements-in-two-binary-search-trees/

中序遍历出来两个有序数组，归并排序一下。

```C++

class Solution {
public:
    vector<int> ans;
    vector<int> getAllElements(TreeNode* root1, TreeNode* root2) {
        vector<int> v1,v2;
        dfs(root1,v1);
        dfs(root2,v2);
        //stl-- std::merge
        // merge(v1.begin(),v1.end(),v2.begin(),v2.end(),back_inserter(ans));
        //手写归并
        int i=0, j=0;
        while(i<v1.size()&&j<v2.size()){
            if(v1[i]<=v2[j]){
                ans.push_back(v1[i]);
                i++;
            }else{
                ans.push_back(v2[j]);
                j++;
            }
        }
        while(i<v1.size()){
            ans.push_back(v1[i]);
            i++;
        }
        while(j<v2.size()){
            ans.push_back(v2[j]);
            j++;
        }
        return ans;
    }


    void dfs(TreeNode* root,vector<int> &ret){
        if(root == NULL){
            return;
        }
        dfs(root->left,ret);
        ret.push_back(root->val);
        dfs(root->right,ret);
    }
};
```

## 230. 二叉搜索树中第K小的元素
https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/

第一版代码就是简单的中序遍历出有序数组，然后返回k-1下标；

看了下大神们的代码，改成了中序遍历然后剪枝。

```C++

class Solution {
public:
    int n = 0;
    int ans = 0;
    int kthSmallest(TreeNode* root, int k) {
        if(root == nullptr){
            return 0;
        }
        search(root,k);
        return ans;
    }

    // void dfs(TreeNode* root,vector<int>& vec){
    //     if(root == nullptr){
    //         return ;
    //     }

    //     dfs(root->left,vec);
    //     vec.push_back(root->val);
    //     dfs(root->right,vec);
    // }

    void search(TreeNode* root, int k){
        if(root == nullptr || n > k){
            return ;
        }
        search(root->left,k);
        n++;
        if(n == k){
            ans = root->val;
        }
        search(root->right,k);
    }
};

```

## 669. 修剪二叉搜索树
https://leetcode-cn.com/problems/trim-a-binary-search-tree/

判断root是否在LR范围内，如果是，就对左右子树递归调用，返回root节点；否则的话，root不在LR范围里，但是它的左子树/右子树还是可能在LR范围里，所以要递归遍历对应的左右子树。返回的是root节点，所以要将返回的root节点赋值给当前root的左右子树。

```C++

class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int L, int R) {
        if(root == NULL){
            return NULL;
        }

        if(root->val < L ){
            return trimBST(root->right,L,R);
        }
        if(root->val>R){
            return trimBST(root->left,L,R);
        }

        root->left = trimBST(root->left,L,R);
        root->right =trimBST(root->right,L,R);

        return root;
    }
};
```

## 653. 两数之和 IV - 输入 BST
https://leetcode-cn.com/problems/two-sum-iv-input-is-a-bst/

两数之和变形题目，原来的想法是对tree做dfs，用map保存rootval，然后遍历map查找合适的节点。。这里要遍历2*N次。

后来看了别人的答案，遍历1*N就可以了，和两数之和1类似。拿到一个数值，就把它放到map里面，然后再查看k-root->val在map里面有没有保存，如果有返回true，否则返回false。

```C++

class Solution {
public:
    unordered_map<int,int> heap;//<remain-val>
    bool findTarget(TreeNode* root, int k) {
        if(root == NULL){
            return false;
        }

        if(heap.count(k-root->val)){
            return true;
        }

        heap[root->val]++;
        return findTarget(root->left,k) || findTarget(root->right,k);
    }

    
};
```

## 面试题 17.12. BiNode
https://leetcode-cn.com/problems/binode-lcci/
和链表插入差不多。一个current指针指向当前节点，中序遍历。

```C++


class Solution {
public:
    TreeNode* cur;
    TreeNode* convertBiNode(TreeNode* root) {
        TreeNode* head = new TreeNode(-1);
        cur = head;
        dfs(root);
        return head->right;
    }

    void dfs(TreeNode* root){
        if(root == NULL){
            return;
        }
        dfs(root->left);
        cur->right = root;
        root->left = NULL;      
        cur = root;
        
        dfs(root->right);
    }
};
```

## 501. 二叉搜索树中的众数
https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/

中序遍历。

用一个变量保存上一次遍历的变量；如果两次结果相等，次数+1,；否则次数归1，将这个变量改为当前root的值。

如果当前次数大于最大次数，清空ans。否则将当前的root的值加入到ans中，更新出现的最大次数。


```C++

class Solution {
public:
    vector<int> findMode(TreeNode* root) {
        vector<int> ans;
        int preval = -1, currentFreq = 0, maxFreq = 0;
        dfs(root,preval,currentFreq,maxFreq,ans);
        return ans;
    }

    void dfs(TreeNode* root, int& preval, int& currentFreq, int& maxFreq, vector<int>& ans){
        if(root == NULL){
            return;
        }

        dfs(root->left,preval,currentFreq,maxFreq,ans);
        if(preval == root->val){
            currentFreq++;
        }else{
            currentFreq = 1;
            preval = root->val; 
        }
        if(currentFreq >= maxFreq){
            if(currentFreq > maxFreq){
                ans.clear();
            }
            ans.push_back(root->val);
            maxFreq = currentFreq;
        }
        dfs(root->right,preval,currentFreq,maxFreq,ans);
    }
};
```
