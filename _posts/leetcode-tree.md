---
title: leetcode-tree[1]
date: 2020-06-23 13:55:54
categories: 
- leetcode
- 树
tags: [leetcode ,树]

---
树系列
<!---more--->


### 节点与其祖先之间的最大差值
<a>https://leetcode-cn.com/problems/maximum-difference-between-node-and-ancestor/</a>

本来首先想到的是后序遍历，但是后来发现想太复杂了，直接前序遍历就可以
老样子，递归三要素。
1、这个函数是干什么的：递归要返回祖先节点和子节点差值的最大值<br>
2、什么时候返回：遍历到叶子结点时候返回<br>
3、返回值是什么：返回祖先节点和子节点差值的最大值，是遍历过程中的最大值与最小值的差值。<br>

```C++

class Solution {
public:
    int maxAncestorDiff(TreeNode* root) {
        int left = maxAncestorDiff(root->left,root->val,root->val);
        int right = maxAncestorDiff(root->right,root->val,root->val);
        return left>right?left:right;
    }
    //返回与祖先节点的最大值
    int maxAncestorDiff(TreeNode* root, int max, int min){
        if(root == NULL){
            return 0;
        }
        //更新最大值和最小值，要求与祖先的最大差值，只要用遍历过程中的最大值与最小值相减就可以
        if(root->val > max){
            max = root->val;
        }
        if(root->val < min){
            min = root->val;
        }
        //叶子节点了，返回过程中最大值与最小值的差值
        if(root->left == NULL && root->right == NULL){
            return max - min;
        }
        //前序遍历
        auto left = maxAncestorDiff(root->left,max,min);
        auto right = maxAncestorDiff(root->right,max,min);
        return left > right? left:right;
    }
};
```
### 完全二叉树的节点个数
<a>https://leetcode-cn.com/problems/count-complete-tree-nodes/</a>

这题暴力遍历就可以。。。当然大神是有其他方法的，先说我这暴力的，就是不停遍历，一个节点+1.
```C++
class Solution {
public:
    int countNodes(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        return countNodes(root->left) + countNodes(root->right) + 1;

    }
};
```
主要思路是利用完全二叉树的性质：左右子树高度相同，左子树是满二叉树；左子树比右子树高，右子树是满二叉树。
满二叉树节点数=2^深度-1。

```python
class Solution(object):
    def countNodes(self, root):
        if not root:
            return 0
        lh, rh = self.__getHeight(root.left), self.__getHeight(root.right)
        if lh == rh:  ## 左右子树高度相同，说明左子树必满 则节点数=左子树节点 + root节点(=1) + 递归找右子树
            return (pow(2, lh) - 1) + 1 + self.countNodes(root.right)
        else:  ## 左子树比右子树高，说明右子树必满 同理
            return (pow(2, rh) - 1) + 1 + self.countNodes(root.left)

    def __getHeight(self, root):
        ret = 0
        while root:
            ret += 1
            root = root.left
        return ret
```

### 填充每个节点的下一个右侧节点指针
<a>https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/</a>

题目要求用常数空间，所以不能用队列层序遍历
还是尽量用递归，前序遍历。<br>
递归函数用于连接节点；<br>
满二叉树
root==null || root->left == null 时返回；<br>
返回root；<br>
然后连接root->left / root->right<br>
后面有一题类似，但是不是满二叉树，所以最好先从右边开始整理

```C++
class Solution {
public:

    Node* connect(Node* root) {
        //leaf node
        if(root == NULL || root->left == NULL)
        {
            return root;
        }
        root->left->next = root->right;
        if(root->next != NULL){
            root->right->next = root->next->left;
        }
        connect(root->right);
        connect(root->left);
        return root;

    }


};
```

### 填充每个节点的下一个右侧节点指针 II
<a>https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/</a><br>
具体内容和上一题有点像，但是不是满二叉树，所以在连接的时候要注意多做判断，同时要从右边开始整理。
```c++

class Solution {
public:
    Node* connect(Node* root) 
    {
        //root==null or root->left dont exits
        if(root == NULL){
            return root;
        }
        if(root->left && root->right){
            root->left->next = root->right;
        }
        //
        if(root->left && root->right == NULL){
            root->left->next = getNext(root->next);
        }
        if(root->right != NULL){
            root->right->next = getNext(root->next);
        }
        //root->left->next depends on root->next, where is the right side of the root node.
        //make sure root->right is always processed.
        connect(root->right);
        connect(root->left);
        return root;
    }

    Node* getNext(Node* root){
        if(root == NULL){
            return NULL;
        }
        if(root->left){
            return root->left;
        }
        if(root->right){
            return root->right;
        }
        if(root->next){
            return getNext(root->next);
        }
        return NULL;
    }
};
```


### 二叉树中第二小的节点
https://leetcode-cn.com/problems/second-minimum-node-in-a-binary-tree/

要注意题目给出的二叉树的性质，根节点是左右节点的较小值，所以根节点是最小的。
主要想法是要想到遍历左右节点，在左右子节点中找到第一个比root大的。<br>
如果当前节点比原根节点大，就可以直接返回，因为他后面的子节点也肯定比他大。<br>
如果左右节点都找到了比原节点大的，返回min(l,r)，否则的话是没找到比原节点大的，返回找到的那个节点。

```C++


class Solution {
public:
    int findSecondMinimumValue(TreeNode* root) {
        if(root==NULL){
            return -1;
        }
        return getFirstBigger(root,root->val);
        
    }

    int getFirstBigger(TreeNode* root,int val){
        if(root == NULL){
            return -1;
        }
        //root->val > val means left node & right node cant be less than val, so min = root->val
        if(root->val>val){
            return root->val;
        }
        //to find l or r node that is larger than val(root->val)
        int l = getFirstBigger(root->left,val);
        int r = getFirstBigger(root->right,val);
        //if l is found & r is found, return min(l,r)
        //else return l or r;
        if(l<0){
            return r;
        }
        if(r<0){
            return l;
        }
        return min(l,r);
    }
};
```
