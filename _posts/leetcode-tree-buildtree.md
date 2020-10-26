---
title: leetcode-tree-buildtree
date: 2020-06-25 17:19:47
categories: 
- leetcode
- 树
tags: [leetcode,树]
---

构造二叉树系列
<!---more--->

### 95. 不同的二叉搜索树 II
https://leetcode-cn.com/problems/unique-binary-search-trees-ii/

从1到n，生成所有可能的二叉树。<br>
从1到n分别作为root节点，root值的左边其他值构造左子树，数组右边构造右子树；<br>
然后将所有的左子树与所有的右子树进行排列组合，构成一棵树。

```C++
class Solution {
public:
    vector<TreeNode*> generateTrees(int n) {
        if(n){
            return generate(1,n);
        }
        return vector<TreeNode*>{};
        
    }

    vector<TreeNode*> generate(int left,int right){
        vector<TreeNode*> ans;
        if(left > right){
            ans.push_back(NULL);
            return ans;
        }
        for(int i=left;i<=right;i++){
            auto leftnodes = generate(left,i-1);
            auto rightnodes = generate(i+1,right);
            for(auto ln:leftnodes){
                for(auto rn:rightnodes){
                    TreeNode* t = new TreeNode(i);
                    t->left = ln;
                    t->right = rn;
                    ans.push_back(t);
                }
            }
        }
        return ans;
    }
};
```

### 654. 最大二叉树
https://leetcode-cn.com/problems/maximum-binary-tree/

构造二叉树的都有点像，就是分左右两个边界，然后递归调用构造。<br>
这道题基本想法就是，在root的左右区间里分别找到最大值的下标，然后将它作为左右子树；最后递归调用，边界分别为(left,index-1)和(idnex+1,right)
```C++

class Solution {
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        return buildMaxTree(nums.begin(),nums.end());
    }

    TreeNode* buildMaxTree(vector<int>::iterator l, vector<int>::iterator r){
        if(l == r){
            return NULL;
        }
        auto maxIndex = max_element(l,r);
        TreeNode* root = new TreeNode(*maxIndex);
        root->left = buildMaxTree(l,maxIndex);
        root->right = buildMaxTree(maxIndex+1,r);
        return root;
    }

    
};
```
### 998. 最大二叉树 II
https://leetcode-cn.com/problems/maximum-binary-tree-ii/
其实这道题感觉和构造二叉树没啥联系，，但是和上面这道最大二叉树有点关系，所以放这里好了。

基本想法就是，如果要插入的节点是大于根节点，那么将插入的节点作为根节点，然后左子树为原树；

如果插入的几点小于根节点，那么就往它的右子树里面去插入，但是在插入的过程，要插入的值可能还是比右子树的根节点小，那么就要递归的往右子树里面去插入，直到插入的节点是大于根节点。所以这就是个递归的过程。

```C++

class Solution {
public:
    TreeNode* insertIntoMaxTree(TreeNode* root, int val) {
        if(root == nullptr){
            return new TreeNode(val);
        }
        if(root->val < val){
            TreeNode* node = new TreeNode(val);
            node->left = root;
            return node;
        }
        else{
            root->right = insertIntoMaxTree(root->right,val);
            return root;
        }
        return nullptr;
    }
};
```


### 105. 从前序与中序遍历序列构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

构造类题目标准模板，buildTree函数，参数是左边界和有边界。

这里根据先序遍历和中序遍历数组确定左子树和右子树的左边界和右边界范围。由先序遍历确定root，然后在中序遍历里面找到root所在index，index左边为左子树范围，右边为右子树范围；回到先序遍历数组里面确定左子树和右子树范围。

```C++

class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return build(preorder,0,preorder.size()-1,inorder,0,inorder.size()-1);
    }

    TreeNode* build(vector<int>& preorder,int leftpre,int rightpre,
    vector<int>& inorder,int leftin,int rightin)
    {
        if(leftpre > rightpre || leftin > rightin){
            return NULL;
        }
        int rootval = preorder[leftpre];
        int mid = rightin;
        for(int i=0;i<=rightin;i++){
            if(inorder[i] == rootval){
                mid = i;
                break;
            }
        }
        auto ret = new TreeNode(rootval);
        //前序左子树范围，中序左子树范围
        ret->left = build(preorder,leftpre+1,leftpre+mid-leftin,inorder,leftin,mid-1);
        ret->right = build(preorder,leftpre+mid-leftin+1,rightpre,inorder,mid+1,rightin);
        return ret;
    }
};

```

### 106. 从中序与后序遍历序列构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/

类似上题

```C++

class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
       return build(inorder,0,inorder.size()-1,postorder,0,postorder.size()-1);
    }

    TreeNode* build(vector<int>& inorder,int leftin,int rightin,vector<int>&postorder,int           leftpost,int rightpost)
    {
        if(leftin>rightin || leftpost > rightpost){
            return NULL;
        }
        int rootval = postorder[rightpost];
        int mid = rightin;
        for(int i=0;i<=rightin;i++){
            if(inorder[i] == rootval){
                mid = i;
                break;
            }
                
        }
        auto ret = new TreeNode(rootval);
        ret->left = build(inorder,leftin,mid-1,postorder,leftpost,leftpost+mid-leftin-1);
        ret->right = build(inorder,mid+1,rightin,postorder,leftpost+mid-leftin,rightpost-1);
        return ret;
    }
};
```

### 889. 根据前序和后序遍历构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/

思想上和上述类似，需要注意的是这里没有中序遍历，所以左子树的root节点需要通过前序遍历的preleft+1获得，所以要注意判断preleft+1<=preright才可以。

```C++


class Solution {
public:
    TreeNode* constructFromPrePost(vector<int>& pre, vector<int>& post) {
        return buildBSTTree(pre,0,pre.size()-1,post,0,post.size()-1);
    }

    TreeNode* buildBSTTree(vector<int>& pre, int preleft, int preright, 
    vector<int>& post, int postleft,int postright)
    {
        if(preleft > preright || postleft > postright){
            return NULL;
        }

        int rootval = pre[preleft];
        TreeNode* root = new TreeNode(rootval);
        int index = preright;
        if(preleft+1 <= preright){
            for(int i=postleft;i<=postright;i++){
                if(pre[preleft+1] == post[i]){
                    index = i;
                    break;
                }
            }
            root->left = buildBSTTree(pre,preleft+1,preleft+index-postleft+1,
            post,postleft,index);

            root->right = buildBSTTree(pre,preleft+index-postleft+2,preright,
            post,index+1,postright-1);
        }
        
        return root;               
    }
};
```

### 1008. 先序遍历构造二叉树

https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/

想法和上面类似，但是因为构造的是二叉搜索树，所以不需要两个遍历数组。

```C++

class Solution {
public:
    TreeNode* bstFromPreorder(vector<int>& preorder) {    
        return buildBSTTree(preorder,0,preorder.size()-1);
    }


    TreeNode* buildBSTTree(vector<int>& preorder,int left ,int right){
        if(left > right){
            return nullptr;
        }
        TreeNode* root = new TreeNode(preorder[left]);
        int index = left;
        while(index <= right && preorder[left] >= preorder[index]){
            index++;
        }

        root->left = buildBSTTree(preorder,left+1,index-1);
        root->right = buildBSTTree(preorder,index,right);
        return root;

    }
};
```
### 449. 序列化和反序列化二叉搜索树
https://leetcode-cn.com/problems/serialize-and-deserialize-bst/

理解题意的意思，将二叉树进行遍历，获取出对应的string；对字符串编码，存成二叉搜索树。以前序遍历为例。获取到首节点以后，区分出左子树和右子树，分别递归生成树。

```C++

class Codec {
public:

    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        if(root == NULL){
            return "*,";
        }
        string s = to_string(root->val) + ',';
        s += serialize(root->left);
        s += serialize(root->right);
        return s;

        
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        if(data.empty()){
            return NULL;
        }
        vector<string> strings;
        split(data,strings);
        vector<int> stringvals;
        for(auto &s:strings){
            if(s == "*"){
                stringvals.push_back(-1);
                continue;
            }
            stringvals.push_back(stoi(s));
        }
        return buildTree(stringvals,0,stringvals.size()-1);
    }

    void split(const string &s, vector<string> &ret, const char &flag = ','){
        stringstream ss(s);
        string item;
        while(getline(ss,item,flag)){
            ret.push_back(item);
        }
    }

    TreeNode* buildTree(vector<int> &vals, int left, int right){
        int val = vals[left];
        if(val == -1){
            return NULL;
        }
        TreeNode* root = new TreeNode(val);
        if(root == NULL || (left >= right)){
            return root;
        }
        int index = left;
        while(index < right && (vals[index] == -1 || vals[index] <= root->val)){
            index++;
        }
        root->left = buildTree(vals,left+1,index-1);
        root->right = buildTree(vals,index,right);
        return root;
    }
};
```

### 894. 所有可能的满二叉树
https://leetcode-cn.com/problems/all-possible-full-binary-trees/

N只有为奇数才可能是满二叉树；将N生成的节点分成两部分，一部分作为左子树一部分作为右子树。

所以对每个小于N的数值i，分别都有可能递归作为左子树的一部分；N-1-i递归作为右子树的一部分；

对于每个左子树和右子树的数组里的节点，进行全排列，创建左右子树。

```C++

class Solution {
public:
    vector<TreeNode*> allPossibleFBT(int N) {
        vector<TreeNode*> ans;
        if(N % 2 == 0){
            return ans;
        }
        if(N == 1){
            TreeNode* t = new TreeNode(0);
            ans.push_back(t);
            return ans;
        }
        else{
            for(int i=1;i<=N-2;i+=2){
                auto left = allPossibleFBT(i);
                auto right = allPossibleFBT(N-1-i);
                for(int j=0;j<left.size();j++){
                    for(int k=0;k<right.size();k++){
                        TreeNode* root = new TreeNode(0);
                        root->left = left[j];
                        root->right = right[k];
                        ans.push_back(root);
                    }
                }
            }
        }
        return ans;
    }
};
```