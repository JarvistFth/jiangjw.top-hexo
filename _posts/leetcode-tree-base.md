---
title: leetcode-tree-base
date: 2020-06-25 17:19:31
categories: 
- leetcode
- 树
tags: [leetcode,树]
---
树的基本特性，例如深度、高度等特性，类型的题目。
<!---more--->
## 对称、相同、翻转
### 相同的树
https://leetcode-cn.com/problems/same-tree/

前序遍历每个节点，如果有一个为null或者节点值不相等就返回false；否则返回true
```C++
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if(p==NULL && q == NULL){
            return true;
        }
        if(q== NULL || p==NULL ){
            return false;
        }
        if(p->val != q->val){
            return false;
        }

        return isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
    }
};
```
### 951. 翻转等价二叉树
https://leetcode-cn.com/problems/flip-equivalent-binary-trees/
给出的函数是用于翻转两个节点的；所以如果两个参数节点有一个为null或者值不相等就返回false；否则返回true。

递归调用的时候，送入的是root1的左子树、root2的右子树以及root1的右子树、root2的左子树，这是翻转一次时需要做的判断；翻转两次做的判断，需要送入的是root1和root2的对应的左右子树（可以理解为相同）


```C++
class Solution {
public:
    bool flipEquiv(TreeNode* root1, TreeNode* root2) {
        if(root1 == NULL && root2 == NULL){
            return true;
        }
        if(root1 == NULL || root2 == NULL || root1->val != root2->val){
            return false;
        }
        return (flipEquiv(root1->left,root2->left) && flipEquiv(root1->right,root2->right))
            || (flipEquiv(root1->left,root2->right) && flipEquiv(root1->right,root2->left));
    }

    
};
```
### 剑指 Offer 26. 树的子结构
https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/

看到题目首先的想法是前序遍历，先对root节点做判断；然后递归调用给出的函数，传入的参数是A的左子树，B以及A的右子树，B。

对root节点做判断要注意，也是一次递归，分别递归左右子树做判断；

如果传入的A为空B不为空，证明A递归到底的时候B还有未处理的节点，返回false；

如果传入的A不为空，B为空了；说明B递归到底的时候，主树A还有子节点，返回true；

如果A和B值不相等，返回false。

递归调用dfs(A->left,B->left)/dfs(A->right,B->right) 对子节点进行判断。
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
### 剑指 Offer 28. 对称的二叉树
判断两个节点是不是相等，需要传入两个参数；对称的要求就是root1->left root2->right && root1->right root2->left 。

如果有一个为空，返回false；如果值不相等，返回false；否则返回true。

```C++
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        return isSymmetric(root,root);
    }

    bool isSymmetric(TreeNode* root1,TreeNode* root2){
        if(root1 == NULL && root2 == NULL){
            return true;
        }
        if(root1 == NULL || root2 == NULL){
            return false;
        }
        if(root1->val != root2->val){
            return false;
        }
        return isSymmetric(root1->left,root2->right) && isSymmetric(root1->right,root2->left);

    }
};
```

### 1379. 找出克隆二叉树中的相同节点
https://leetcode-cn.com/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/

同时递归遍历，如果找到target节点，返回当前的cloned；如果节点数相同还要判断子树，这里就不写了。
```C++

class Solution {
public:
    TreeNode* getTargetCopy(TreeNode* original, TreeNode* cloned, TreeNode* target) {
        if(original == NULL){
            return NULL;
        }

        if(original == target){
            return cloned;
        }

        TreeNode* left = getTargetCopy(original->left,cloned->left,target);
        return left!=NULL?left:getTargetCopy(original->right,cloned->right,target);
    }
};
```

## 深度
865. 具有所有最深结点的最小子树

https://leetcode-cn.com/problems/most-frequent-subtree-sum/

递归检查深度，往深度大的那边递归调用返回子树。如果两边深度一样，那这个就是结果。


```C++

class Solution {
public:
    int maxDepth = INT_MIN;
    TreeNode* subtreeWithAllDeepest(TreeNode* root) {
        if(root == NULL){
            return NULL;
        }

        int left = dfs(root->left);
        int right = dfs(root->right);
        if(left == right){
            return root;
        }
        else if(left > right){
            return subtreeWithAllDeepest(root->left);
        }else{
            return subtreeWithAllDeepest(root->right);
        }
    }

    int dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        return max(dfs(root->left),dfs(root->right)) + 1;
    }
};

```

### 559. N叉树的最大深度
 https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/

 对每个子树递归遍历，记录其每个子树的最大深度，递归函数返回其最大深度+1，
 ```C++

class Solution {
public:
    int maxDepth(Node* root) {
        return dfs(root);
    }

    int dfs(Node* root){
        if(root == nullptr){
            return 0;
        }
        int depth = 0;
        for(auto n:root->children){
            depth = max(depth,dfs(n));
        }
        return depth+1;
    }
};
 ```



## 高度
### 面试题 04.04. 检查平衡性
https://leetcode-cn.com/problems/check-balance-lcci/

平衡的定义是两棵子树高度差不超过1，所以通过递归，计算左右子树的高度，如果高度差大于1，返回false；递归函数返回的是树的高度（左右子树的最大值+1）
```C++
class Solution {
public:
     bool ret = true;
    bool isBalanced(TreeNode* root) {
        if(root == NULL){
            return ret;
        }
        getHeight(root);
        return ret;
    }

    int getHeight(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int leftHeight = getHeight(root->left);
        int rightHeight = getHeight(root->right);
        if(abs(leftHeight - rightHeight) > 1){
            ret = false;
        }
        return max(leftHeight,rightHeight) + 1;
    }
};
```
## 直径
### 543. 二叉树的直径
https://leetcode-cn.com/problems/diameter-of-binary-tree/
二叉树的直径，和高度类似，分别求左右子树的高度，最大直径就是左右子树高度之和；由于直径可能不过root，所以需要对每个root都求一下，然后用一个变量存储目前的直径最大值用于比较；最后返回真·最大值。
```C++

class Solution {
public:
    int ans = 0;
    int diameterOfBinaryTree(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        maxHeight(root);
        return ans;
    }

    int maxHeight(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int left = maxHeight(root->left);
        int right = maxHeight(root->right);
        ans = max(ans,(right+left));
        return 1+max(left,right);
    }
};
```
## 合并
### 617. 合并二叉树
https://leetcode-cn.com/problems/merge-two-binary-trees/

如果两个节点A/B都为空，返回null

如果其中一个节点为空，返回另一个节点；

否则两个节点的值相加；然后递归调用函数，传入左子树和右子树。

```C++

class Solution {
public:
    
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if(t1 == NULL && t2 == NULL){
            return NULL;
        }
        if(t1 == NULL){
            return t2;
        }
        if(t2 == NULL){
            return t1;
        }
        t1->val += t2->val;
        t1->left = mergeTrees(t1->left,t2->left);
        t1->right = mergeTrees(t1->right,t2->right);
        return t1;
    }
};
```

## 求和

### 979. 在二叉树中分配硬币
https://leetcode-cn.com/problems/distribute-coins-in-binary-tree/

官方题解比较好 这道题还挺难理解

https://leetcode-cn.com/problems/distribute-coins-in-binary-tree/solution/zai-er-cha-shu-zhong-fen-pei-ying-bi-by-leetcode/
```C++
class Solution {
public:
    int ans = 0;
    int distributeCoins(TreeNode* root) {
        getCoinNum(root);
        return ans;
    }

    //返回分配金币数目
    int getCoinNum(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int left = getCoinNum(root->left);
        int right = getCoinNum(root->right);

        ans += abs(left) + abs(right);
        //left or right could be less than 0, means coins needs move from root to left(right)
        //
        return left+right+root->val-1;
    }
};
```

### 124. 二叉树中的最大路径和
https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/

对于任意一个节点, 如果最大和路径包含该节点, 那么只可能是两种情况:
        1. 其左右子树中所构成的和路径值较大的那个加上该节点的值后向父节点回溯构成最大路径
        2. 左右子树都在最大路径中, 加上该节点的值构成了最终的最大路径


```C++

class Solution {
public:
    int ans = INT_MIN;
    int maxPathSum(TreeNode* root) {
        getMax(root);
        return ans;
    }

    int getMax(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        auto left = max(0,getMax(root->left));
        auto right = max(0,getMax(root->right));
        ans = max(ans,left + right + root->val);
        return max(left,right)  + root->val;
    }
};
```

### 508. 出现次数最多的子树元素和
https://leetcode-cn.com/problems/most-frequent-subtree-sum/<br>

用map记录子树出现的次数和其对应的子树和。由于次数是可以逐渐增加的，所以作为value，key则是用元素和sum。遍历map，如果找到出现的次数与最大次数相等，则返回key。所以用一个全局变量存储出现最多的次数，子树和就是root->val + dfs(root->left) + dfs(root->right)。

```C++

class Solution {
public:
    vector<int> ans;
    int maxFrequency = 0;
    // map:('sum','frequency') 
    //if map[i] == maxfrequency, add sum to ans; 

    unordered_map<int,int> sumFreqMap;
    vector<int> findFrequentTreeSum(TreeNode* root) {
        dfs(root);
        for(auto& [s,f]:sumFreqMap){
            if(f == maxFrequency){
                ans.push_back(s);
            }
        }
        return ans;
    }

    int dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }

        int left = dfs(root->left);
        int right = dfs(root->right);
        int sum = left + right + root->val;

        maxFrequency = max(maxFrequency,++sumFreqMap[sum]);
        return sum;
    }
};
```

### 129. 求根到叶子节点数字之和
https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/
先序遍历，每次传入当前的数字和作为参数。如果碰到叶子节点，就返回当前的和加节点的值，否则递归调用，传入当前和*10

```C++

class Solution {
public:
    int sumNumbers(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        return dfs(root,0);

    }

    int dfs(TreeNode* root, int sum){
        if(root == NULL){
            return 0;
        }
        if(!root->left && !root->right){
            return sum+root->val;
        }
        sum += root->val;
        return dfs(root->left,sum*10) + dfs(root->right,sum*10);

    }
};
```

### 112. 路径总和
https://leetcode-cn.com/problems/path-sum/

dfs,每次dfs传入一个sum，然后将sum-=root->val，如果减完发现sum为0，并且是叶子节点，就符合要求。

```C++

class Solution {
public:
    bool ans = false;
    bool hasPathSum(TreeNode* root, int sum) {
        dfs(root,sum);
        return ans;
    }

    void dfs(TreeNode* root,int sum){
        if(root == NULL){
            return;
        }
        sum -= root->val;
        if(sum==0 && root->left == NULL && root->right == NULL){
            ans = true;
        }
        dfs(root->left,sum);
        dfs(root->right,sum);
    }
};
```

### 面试题 04.12. 求和路径
https://leetcode-cn.com/problems/paths-with-sum-lcci/
和上题类似，但是要注意一点的是，它不要求从根节点到叶子节点，所以sum==0时不需要判断是否是叶子结点，同时还需要用原函数递归一下root->left root->right

```C++

class Solution {
public:
    int ans = 0;
    int pathSum(TreeNode* root, int sum) {
        if(root == NULL){
            return 0;
        }
        dfs(root,sum);
        pathSum(root->left,sum);
        pathSum(root->right,sum);
        return ans;
    }

    void dfs(TreeNode* root, int sum){
        if(root == NULL){
            return;
        }

        sum-= root->val;
        if(sum == 0 ){
            ans++;
        }
        dfs(root->left,sum);
        dfs(root->right,sum);
    }
};
```

### 1339. 分裂二叉树的最大乘积
https://leetcode-cn.com/problems/maximum-product-of-splitted-binary-tree/

分成两部分的二叉树的和也可以分为两部分，所以用一个vector存部分1的和，部分2的和就等于总和减部分1的和。然后保留部分1的和与部分2的和的最大值就可以了。


```C++

class Solution {
public:
    vector<long long> vec;
    int mod = 1e9 + 7;
    int maxProduct(TreeNode* root) {
        if(root == NULL){
            return 0;
        }
        long ans = 0;
        long long sum = dfs(root);
        for(auto i:vec){
            long multi = ((sum - i) * i);
            ans = max(multi,ans);
        }
        return (int)(ans % mod);
    }

    long long dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        long long sum = root->val + dfs(root->left) + dfs(root->right);
        vec.push_back(sum);
        return sum;
    }
};
```


## 路径
### 1372. 二叉树中的最长交错路径
https://leetcode-cn.com/problems/longest-zigzag-path-in-a-binary-tree/

遍历左右结点，如果是左结点，返回右边深度+1；否则反之；再和全局变量比较。

```C++

class Solution {
public:
    int ans = INT_MIN;
    int longestZigZag(TreeNode* root) {
        dfs(root,true);
        return ans;
    }

    int dfs(TreeNode* root,bool isLeft){
        if(root == NULL){
            return 0;
        }
        int left = dfs(root->left,true);
        int right = dfs(root->right,false);
        int ret = max(left,right);
        if(ret > ans){
            ans = ret;
        }
        if(isLeft){
            return right+1;
        }else{
            return left+1;
        }
    }
};
```

## 节点数
### 1145. 二叉树着色游戏

https://leetcode-cn.com/problems/binary-tree-coloring-game/

围观了大神的写法。。胜利条件是可着色节点数量大于一半以上，着色后将节点分为三个区域，左节点，右节点，其他节点。所以最大可着色节点数量等于三者的最大值，其他节点数量为n-l-r-1 。所以问题回到计算节点数量的问题上。
```C++

class Solution {
public:
    bool btreeGameWinningMove(TreeNode* root, int n, int x) {
        if(root == NULL){
            return false;
        }

        if(root->val == x){
            int left = dfs(root->left);
            int right = dfs(root->right);
            int others = n - left - right - 1;
            int maxNum = max(max(left,right),others);
            if(maxNum > n/2){
                return true;
            }
        }
        return btreeGameWinningMove(root->left,n,x) || btreeGameWinningMove(root->right,n,x);
    }

    int dfs(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int left = dfs(root->left);
        int right = dfs(root->right);
        return left + right + 1;
    }
};
```

### 501. 二叉搜索树中的众数
https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/

统计每个节点的值，找到某个值出现最多的节点。这是简单思路。不利用额外空间，就是按照中序遍历遍历二叉树，这样可以得到一个升序序列，这时候可以用一个值记录上次遍历的节点值，判断其是否和当前的相等，如果相等，出现次数加1，否则出现次数归1，然后用一个值记录最大的出现次数，进行对比；如果大于最大值，就清空原vector，并且将当前的值加入vector。
```C++

class Solution {
public:
    vector<int> ans;
    int preval;
    int currentFreq = 0 ,maxFreq = 0;
    vector<int> findMode(TreeNode* root) {
        dfs(root);
        return ans;
    }

    void dfs(TreeNode* root){
        if(root == NULL){
            return;
        }

        dfs(root->left);
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
            ans.push_back(preval);
            maxFreq = currentFreq;
        }
        dfs(root->right);


    }
};
```


