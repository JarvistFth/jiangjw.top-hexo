---
title: leetcode-树-分类
date: 2020-06-25 13:53:07
categories: 
- leetcode
- 树
tags: [leetcode ,树]
keywords: [leetcode,树]


---
二叉树基本遍历方法，同时对二叉树类型的题目进行了大致的分类
<!---more--->

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

## 树问题的分类



### 1、基本特性
#### 1-1：对称、相同、翻转：
 [100\. 相同的树](https://leetcode-cn.com/problems/same-tree/)<br>
 [951. 翻转等价二叉树](https://leetcode-cn.com/problems/flip-equivalent-binary-trees/)<br>
 [面试题26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)<br>
 [面试题28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)<br>
 [1379. 找出克隆二叉树中的相同节点](https://leetcode-cn.com/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/)<br>


#### 1-2：深度
[865. 具有所有最深结点的最小子树](https://leetcode-cn.com/problems/most-frequent-subtree-sum/)<br>
[559. N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)<br>

#### 1-3：高度：
[面试题 04.04](https://leetcode-cn.com/problems/check-balance-lcci/)：检查平衡性<br>

#### 1-4：直径：
[#543](https://leetcode-cn.com/problems/diameter-of-binary-tree/)：二叉树的直径<br>

#### 1-5：合并：
 [617\. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)<br>


#### 1-6 求和
[124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)<br>
[979. 在二叉树中分配硬币](https://leetcode-cn.com/problems/distribute-coins-in-binary-tree/)<br>
[508. 出现次数最多的子树元素和](https://leetcode-cn.com/problems/most-frequent-subtree-sum/)<br> 
[129. 求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/)<br>
[面试题 04.12. 求和路径](https://leetcode-cn.com/problems/paths-with-sum-lcci/)<br>
[112. 路径总和](https://leetcode-cn.com/problems/path-sum/)<br>
[1339. 分裂二叉树的最大乘积](https://leetcode-cn.com/problems/maximum-product-of-splitted-binary-tree/)<br>



#### 1-7 路径
[1372. 二叉树中的最长交错路径](https://leetcode-cn.com/problems/longest-zigzag-path-in-a-binary-tree/)<br>

#### 1-8 节点数
[1145. 二叉树着色游戏](https://leetcode-cn.com/problems/binary-tree-coloring-game/)<br>
[501. 二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)<br>



### 2、构造二叉树
 [95\. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)<br>
 [654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)<br>
 [998. 最大二叉树 II](https://leetcode-cn.com/problems/maximum-binary-tree-ii/)<br>
 [从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)<br>
 [从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)<br>
 [根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)<br>
 [先序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)<br>
 [449. 序列化和反序列化二叉搜索树](https://leetcode-cn.com/problems/serialize-and-deserialize-bst/)
[894. 所有可能的满二叉树](https://leetcode-cn.com/problems/all-possible-full-binary-trees/)<br>

 
### 3、节点关系
#### 3-1：祖先节点：
[#面试题 04.08](https://leetcode-cn.com/problems/first-common-ancestor-lcci/)：首个共同祖先<br>
#### 3-2：兄弟节点：
[993. 二叉树的堂兄弟节点](https://leetcode-cn.com/problems/cousins-in-binary-tree/)<br>
#### 3-3：子节点：


### 4：遍历问题：
#### 4-1:前序遍历：
[1022：从根到叶的二进制数之和](https://leetcode-cn.com/problems/sum-of-root-to-leaf-binary-numbers/)<br>
[606\. 根据二叉树创建字符串](https://leetcode-cn.com/problems/construct-string-from-binary-tree/)<br>
[652. 寻找重复的子树](https://leetcode-cn.com/problems/find-duplicate-subtrees/)<br>
[1315. 祖父节点值为偶数的节点和](https://leetcode-cn.com/problems/sum-of-nodes-with-even-valued-grandparent/)<br>
[971. 翻转二叉树以匹配先序遍历](https://leetcode-cn.com/problems/flip-binary-tree-to-match-preorder-traversal/)<br>
[1448. 统计二叉树中好节点的数目](https://leetcode-cn.com/problems/count-good-nodes-in-binary-tree/)<br>
[1261. 在受污染的二叉树中查找元素](https://leetcode-cn.com/problems/find-elements-in-a-contaminated-binary-tree/)<br>




#### 4-2：中序遍历：
[897. 递增顺序查找树](https://leetcode-cn.com/problems/increasing-order-search-tree/)<br>
#### 4-3：后序遍历：
[563\. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)<br>
[814. 二叉树剪枝](https://leetcode-cn.com/problems/binary-tree-pruning/)
[1110. 删点成林](https://leetcode-cn.com/problems/delete-nodes-and-return-forest/)<br>
[988. 从叶结点开始的最小字符串](https://leetcode-cn.com/problems/smallest-string-starting-from-leaf/)<br>
[1325. 删除给定值的叶子节点](https://leetcode-cn.com/problems/delete-leaves-with-a-given-value/)<br>
[1123. 最深叶节点的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-deepest-leaves/) <br>
[872. 叶子相似的树](https://leetcode-cn.com/problems/leaf-similar-trees/)<br>
[114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)<br>


#### 4-4：层次遍历
[623. 在二叉树中增加一行](https://leetcode-cn.com/problems/add-one-row-to-tree/)<br>
[199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)<br>
[958. 二叉树的完全性检验](https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/)<br>
[662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)<br>
[515. 在每个树行中找最大值](https://leetcode-cn.com/problems/find-largest-value-in-each-tree-row/)<br>
[919. 完全二叉树插入器](https://leetcode-cn.com/problems/complete-binary-tree-inserter/)<br>





#### 4-5：特殊遍历

### 5、二叉搜索树
[#面试题 04.06](https://leetcode-cn.com/problems/successor-lcci/)  面试题 04.06. 后继者<br>
[#面试题 04.05](https://leetcode-cn.com/problems/legal-binary-search-tree-lcci/)：合法二叉搜索树<br>
[验证二叉搜索树]
(https://leetcode-cn.com/problems/validate-binary-search-tree/)<br>

[#530：二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)<br>
[#538：二叉搜索树转换成累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)<br>
 [#173\. 二叉搜索树迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/)<br>
 [938. 二叉搜索树的范围和](https://leetcode-cn.com/problems/range-sum-of-bst/)<br>
 [450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)<br>
 [501. 二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)<br>
 [701. 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)<br>
 [1305. 两棵二叉搜索树中的所有元素](https://leetcode-cn.com/problems/all-elements-in-two-binary-search-trees/)<br>
 [230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)<br>
 [669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)<br>
 [653. 两数之和 IV - 输入 BST](https://leetcode-cn.com/problems/two-sum-iv-input-is-a-bst/)<br>
 [面试题 17.12. BiNode](https://leetcode-cn.com/problems/binode-lcci/)<br>

 ### 6、双重递归

[#1367. 二叉树中的列表](https://leetcode-cn.com/problems/linked-list-in-binary-tree/)<br>

[剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/) 

### 7、回溯
[1457. 二叉树中的伪回文路径](https://leetcode-cn.com/problems/pseudo-palindromic-paths-in-a-binary-tree/)<br>