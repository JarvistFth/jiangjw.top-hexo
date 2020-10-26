---
title: leetcode-tree-path
date: 2020-06-25 17:21:51
categories: 
- leetcode
- 树
tags: [leetcode ,树]
---

树的路径问题以及遍历问题
<!---more--->
## 前序遍历

### N叉树的前序遍历
https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/

经典前序遍历
递归
```C++

class Solution {
public:
    vector<int> ret;
    vector<int> preorder(Node* root) {
        if(root == nullptr){
            return {};
        }
        ret.push_back(root->val);
        for(auto c:root->children){
            preorder(c);
        }
        return ret;

        
    }
};
```
非递归，先将右节点入栈，访问就先从左节点开始了。
```C++
class Solution {
public:
    vector<int> ret;
    stack<Node*> s;
    vector<int> preorder(Node* root) {
        if(root == nullptr){
            return {};
        }
        //
        // ret.push_back(root->val);
        // for(auto c:root->children){
        //     preorder(c);
        // }
        // return ret;

        s.push(root);
        while(!s.empty()){
            Node* node = s.top();
            s.pop();
            ret.push_back(node->val);
            for(int i=node->children.size()-1;i>=0;i--){
                s.push(node->children[i]);
            }
        }
        return ret;

        
    }
};
```
### 971. 翻转二叉树以匹配先序遍历
https://leetcode-cn.com/problems/flip-binary-tree-to-match-preorder-traversal/
前序遍历，如果当前root节点和voyage的当前下标的值不相等，清空vector返回-1；如果有左子树，并且左子树的值和voyage下一个值不相等，翻转（左右子树交换遍历顺序，先右后左），否则正常遍历。

```C++

class Solution {
public:
    int index = 0;
    vector<int> ans;
    vector<int> flipMatchVoyage(TreeNode* root, vector<int>& voyage) {
        dfs(root,voyage);
        return ans;
    }

    void dfs(TreeNode* root, vector<int>& voyage){
        if(root == NULL){
            return;
        }
        if(root->val != voyage[index++]){
            ans.clear();
            ans.push_back(-1);
            return;
        }
        if(index<voyage.size() && root->left && root->left->val != voyage[index] ){
            ans.push_back(root->val);
            dfs(root->right,voyage);
            dfs(root->left,voyage);
        }
        else{
            dfs(root->left,voyage);
            dfs(root->right,voyage);
        }
    }
};
```


### 1022. 从根到叶的二进制数之和
https://leetcode-cn.com/problems/sum-of-root-to-leaf-binary-numbers/

前序遍历，如果是叶子结点，返回sum；如果一侧空，遍历另一侧；两侧都有就两侧遍历，返回的是两侧的和。
```C++

class Solution {
public:
    int sumRootToLeaf(TreeNode* root) {
        return dfs(root,0);
        
    }

    //返回当前root是叶节点时表示的和 sum--当前root的和
    int dfs(TreeNode* root,int sum){
        if(root == NULL){
            return 0;
        }
        sum = sum*2 + root->val;
        //叶子结点直接返回sum
        if(root->left==NULL && root->right == NULL){
            return sum;
        }
        //左边空，只编历右边
        if(root->left == NULL){
            return dfs(root->right,sum);
        }
        //右边空，只遍历左边
        if(root->right == NULL){
            return dfs(root->left,sum);
        }
        //两边都有，遍历左右，返回的是左右节点为root时表示的和，
        return dfs(root->right,sum) + dfs(root->left,sum);
    }
};
```

### 606. 根据二叉树创建字符串
https://leetcode-cn.com/problems/construct-string-from-binary-tree/

前序遍历，如果是叶子节点，直接返回root->val；如果没有右子树，括号会少一对；否则递归调用左子树和右子树。

```C++

class Solution {
public:
    string tree2str(TreeNode* root) {
        if(root != NULL){
            if(root->left == NULL && root->right == NULL){
                return to_string(root->val) + "";
            }
            if(root->left != NULL && root->right ==NULL){
                return to_string(root->val) + "(" + tree2str(root->left) + ")";
            }
            return to_string(root->val) + "(" + tree2str(root->left) + ")(" 
            + tree2str(root->right) + ")";

        }
        return "";   
    }
};
```
### 652. 寻找重复的子树
https://leetcode-cn.com/problems/find-duplicate-subtrees/

将root->val + dfs(root->left) + dfs(root->right)， 送入map，map中保存每个子树的数值。如果此时map中统计出次数为1，就说明之前map中已经有这个子树，将此时子树的值放入ans中。
要注意遍历的时候子树的所有值为to_string(root->val)+' '+dfs(root->left)+' '+dfs(root->right)，这是以root为根节点的所有子树的值，只有所有子树相同，才认为是重复。
```C++

class Solution {
public:
    vector<TreeNode*> ans;
    map<string,int> m;
    vector<TreeNode*> findDuplicateSubtrees(TreeNode* root) {
        if(root == nullptr){
            return ans;
        }
        dfs(root);
        return ans;
    }

    string dfs(TreeNode* root){
        if(root == nullptr){
            return "";
        }
        string rootstring = to_string(root->val) + ' ' + dfs(root->left) + ' ' + dfs(root->right);
        if(m[rootstring] == 1){
            ans.push_back(root);
        }
        m[rootstring]++;
        return rootstring;
    }
};
```

### 1315. 祖父节点值为偶数的节点和
https://leetcode-cn.com/problems/sum-of-nodes-with-even-valued-grandparent/

直接dfs，传入grandparent节点以及parent节点；根据grandparent节点，判断祖父节点数值是否为偶数，如果是，就累加。

```C++

class Solution {
public:
    int sumEvenGrandparent(TreeNode* root) {
        return dfs(NULL,NULL,root);
    }
    
    int dfs(TreeNode* grandParent, TreeNode* parent, TreeNode* root){
        if(root == NULL){
            return 0;
        }
        int sum = 0;
        if(grandParent != NULL && (grandParent->val % 2) == 0){
            sum += root->val;
        }
        sum += dfs(parent,root,root->left);
        sum += dfs(parent,root,root->right);
        return sum;
    }
};

```

### 1448. 统计二叉树中好节点的数目
https://leetcode-cn.com/problems/count-good-nodes-in-binary-tree/

看到首先想法是dfs，每次dfs维护一个最大值max，如果root->val大于等于max，说明下层的节点值比上层最大值大，这个节点是好节点。这时候答案+1，并且更新最大值，然后更新dfs。
```C++

class Solution {
public:
    int ans = 1;
    int goodNodes(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        dfs(root->left,root->val);
        dfs(root->right,root->val);
        return ans;

        
    }


    void dfs(TreeNode* root, int max){
        if(root == nullptr){
            return;
        }
        if(root->val >= max){
            max = root->val;
            ans++;
        }
        dfs(root->left,max);
        dfs(root->right,max);

    }
};
```

### 1261. 在受污染的二叉树中查找元素
https://leetcode-cn.com/problems/find-elements-in-a-contaminated-binary-tree/
直接暴力解决了。。
```C++

class FindElements {
public:
    FindElements(TreeNode* root) {
        if(root != NULL){
            if(root->val == -1){
                root->val = 0;
            }
            if(root->left){
                root->left->val  = root->val * 2 +1;
            }
            if(root->right){
                root->right->val = root->val * 2 + 2;
            }
            FindElements(root->left);
            FindElements(root->right);
        }
        this->root = root;

    }
    
    bool find(int target) {
        return find(target,this->root);

    }

    bool find(int target, TreeNode* node){
        //node->val >target , go deeper then node->val gets larger, so return false
        if(node == NULL || node->val > target){
            return false;
        }
        if(node->val == target){
            return true;
        }
        return find(target,node->left) || find(target,node->right);
    }
private:
    TreeNode* root;
};
```



## 中序遍历
### 897. 递增顺序查找树
https://leetcode-cn.com/problems/increasing-order-search-tree/

中序遍历，用一个全局变量保存上次遍历的root节点的值。
```C++
class Solution {
public:
    TreeNode* head = new TreeNode(0);
    TreeNode* pre = head;
    TreeNode* increasingBST(TreeNode* root) {
        if(root == NULL){
            return NULL;
        }

    auto left = increasingBST(root->left);
    pre->right = new TreeNode(root->val);
    pre = pre->right;
    auto right = increasingBST(root->right);
    return head->right;
    }
};
```

## 后序遍历
### 1110. 删点成林
https://leetcode-cn.com/problems/delete-nodes-and-return-forest/

这道题，要删除结点，所以不能用先序，因为删了，下面就没办法遍历了。所以用后序遍历。如果遍历到的root结点值在vector里，就把左右子树放入ans，并且将root置空。

```C++

class Solution {
public:
    vector<TreeNode*> ans;
    vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
        dfs(root,to_delete);
        if(root){
            ans.push_back(root);
        }
        return ans;

    }

    void dfs(TreeNode* &root,vector<int>& to_delete){
        if(root == NULL){
            return;
        }

        dfs(root->left,to_delete);
        dfs(root->right,to_delete);
        //root->val belongs to to_delete set?root=null:return;
        if(find(to_delete.begin(),to_delete.end(),root->val) != to_delete.end()){
            if(root->left){
                ans.push_back(root->left);
            }
            if(root->right){
                ans.push_back(root->right);
            }
            root = NULL;
        }
    }
};
```

### 988. 从叶结点开始的最小字符串
https://leetcode-cn.com/problems/smallest-string-starting-from-leaf/

官方题解是回溯+先序遍历，但是我看到题目首先想到的是后序，因为他是从叶子结点开始的。想法就是如果是叶子结点，返回root->val对应的string，如果左右子树只有一个，继续遍历；如果左右子树都有，就遍历他们两个，返回他们中值较小的那个；返回值是做好的字符串，所以需要将当前节点对应的字符值传入作为参数。
```C++

class Solution {
public:
    string ans;
    string smallestFromLeaf(TreeNode* root) {
        if(root == NULL){
            return "";
        }
        return dfs(root,ans);
    }

    string dfs(TreeNode* root,string before){
        if(root == NULL){
            return 0;
        }
        if(root->left == NULL && root->right == NULL){
            return (char)('a'+root->val) + before;
        }
        if(root->left == NULL && root->right != NULL){
            return dfs(root->right,(char)(root->val+'a')+before);
        }
        if(root->left != NULL && root->right == NULL){
            return dfs(root->left,(char)(root->val+'a')+before);
        }
        if(root->left && root->right){
            string left = dfs(root->left,(char)(root->val+'a')+before);
            string right = dfs(root->right,(char)(root->val+'a')+before);
            if(left<right){
                return left;
            }else{
                return right;
            }

        }
        return "";
    }
};

```
### 1325. 删除给定值的叶子节点
https://leetcode-cn.com/problems/delete-leaves-with-a-given-value/

看到是叶子节点首先想到是后序遍历。后序遍历，如果是val值就返回null，否则返回root；因为返回的是叶子结点，所以要将当前的左右子节点赋值为后序遍历的返回值。

```C++

class Solution {
public:
    TreeNode* removeLeafNodes(TreeNode* root, int target) {
        if(root == nullptr){
            return nullptr;
        }

        root->left = removeLeafNodes(root->left,target);
        root->right = removeLeafNodes(root->right,target);

        if(root->left == nullptr && root->right == nullptr && root->val == target){
            return nullptr;
        }
        return root;
    }
};
```

### 1123. 最深叶节点的最近公共祖先
https://leetcode-cn.com/problems/lowest-common-ancestor-of-deepest-leaves/

看到叶子节点应该先想到后序遍历，需要计算深度。如果后序遍历左右深度相等，证明当前左右子树的叶子节点的两个深度相等，所以返回当前的root节点；否则的话对深度大的那一边进行递归。
```C++

class Solution {
public:
    TreeNode* lcaDeepestLeaves(TreeNode* root) {
        if(root == NULL){
            return NULL;
        }

        int left = getDepth(root->left);
        int right = getDepth(root->right);
        if(left == right){
            return root;
        }
        return left>right?lcaDeepestLeaves(root->left):lcaDeepestLeaves(root->right);
    }

    int getDepth(TreeNode* root){
        if(root == NULL){
            return 0;
        }
        auto left = getDepth(root->left);
        auto right = getDepth(root->right);
        return max(left,right)+1;
    }
};
```

### 872. 叶子相似的树
https://leetcode-cn.com/problems/leaf-similar-trees/
后序遍历放到两个vector里面进行比较。

```C++

class Solution {
public:
    vector<int> v1,v2;
    bool leafSimilar(TreeNode* root1, TreeNode* root2) {
        dfs(root1,v1);
        dfs(root2,v2);
        if(v1 == v2){
            return true;
        }
        return false;
    }

    void dfs(TreeNode* root, vector<int>& v){
        if(root == NULL){
            return ;
        }

        dfs(root->left,v);
        dfs(root->right,v);
        if(root->left == NULL && root->right == NULL){
            v.push_back(root->val);
        }
    }
};
```

### 114. 二叉树展开为链表
https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/ 

在还没操作节点右子树前，不能破坏该节点的右子树指向。所以采用后序遍历。

```C++

class Solution {
public:
    void flatten(TreeNode* root) {
        if(root == nullptr){
            return ;
        }
        //撸直左子树，左子树只有right
        flatten(root->left);
        //撸直右子树，右子树只有right
        flatten(root->right);
        //合并撸直后的左右子树，令一个中间值为right，将right指向left，然后遍历right到底，将right指向中间值
        TreeNode* pre = root->right;
        root->right = root->left;
        root->left = nullptr;
        while(root->right != nullptr){
            root = root->right;
        }
        root->right = pre;
        
    }
};
```





## 层序遍历
### 623. 在二叉树中增加一行 
https://leetcode-cn.com/problems/add-one-row-to-tree/

这道题层序遍历比较直观，递归代码比较简单。层序的想法是将每一层的节点入队列，然后当前层的计数器++，判断计数器与d是否相等，相等的话新建节点，调整节点关系；否则继续将当前节点的子节点入队列。

递归调用有两种写法，一种代码量比较少，但是不好比较或者理解思路；另一种比较好理解，照旧d==1时，返回特殊情况；然后创建新函数，传入level作为参数，每次递归传入的level+1，判断level与d是否相等，相等就插入节点；否则递归调用。开始调用时要注意是从level=2开始，因为这里的level是要插入的行数，d==1外的情况，从第2行开始插入。

非递归
```C++
class Solution {
public:
    //d-1 root
    TreeNode* addOneRow(TreeNode* root, int v, int d) {
        if(d == 1){
            TreeNode* newRoot = new TreeNode(v);
            newRoot->left = root;
            return newRoot;
        }

        queue<TreeNode*> rowQueue;
        rowQueue.push(root);
        int currentLevel = 1;
        bool insertFinished = false;
        while(!rowQueue.empty() 
        && !insertFinished
        ){
            int size = rowQueue.size();
            for(int i=0;i<size;i++){
                TreeNode* node = rowQueue.front();
                rowQueue.pop();
                if(currentLevel == d-1){
                    TreeNode* tmp = node->left;
                    node->left = new TreeNode(v);
                    node->left->left = tmp;
                    tmp = node->right;
                    node->right = new TreeNode(v);
                    node->right->right = tmp;
                    insertFinished = true;
                }
                else{
                    if(node->left){
                        rowQueue.push(node->left);
                    }
                    if(node->right){
                        rowQueue.push(node->right);
                    }
                }
            }
            currentLevel++; 
        }
        return root;
    }
};
```

递归调用
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
    //d-1 root
    TreeNode* addOneRow(TreeNode* root, int v, int d) {
        if(!root){
            return NULL;
        }

        if(d == 1){
            TreeNode* newRoot = new TreeNode(v);
            newRoot->left = root;
            return newRoot;
        }
        return add(root,v,d,2);
    }

    TreeNode* add(TreeNode* root,int v, int d, int level){
        if(root == NULL){
            return NULL;
        }
        if(level == d){
            TreeNode* nLeft = new TreeNode(v);
            TreeNode* nRight = new TreeNode(v);

            nLeft->left = root->left;
            nRight->right = root->right;
                
            root->left = nLeft;
            root->right = nRight;
                
            // return root;
        }else{
            add(root->left,v,d,level+1);
            add(root->right,v,d,level+1);
        }
        return root;
    }
};
```

### 199. 二叉树的右视图
https://leetcode-cn.com/problems/binary-tree-right-side-view/

这题简单，层序遍历，ans保留每一层最右边的节点就可以了。

```C++

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
### 958. 二叉树的完全性检验
https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/

这道题用了层次遍历的想法，但是在具体操作的时候有些地方不太一样，一个是它不需要对每层的每个节点分清楚；具体来说就是在队列里面只需要保存遍历过的节点，将它的子树入队就行了，不需要将每个节点的层次分清楚（少了个for循环）。在判断的时候，如果左子树是空右子树不空，就是false；如果编历过程碰到两次以上的空节点并且此时node为非叶子结点，也为false。所以用一个flag标记之前是否已经碰到过空节点。

```C++

class Solution {
public:
    bool firstNull = false;
    bool isCompleteTree(TreeNode* root) {
        if(root == NULL){
            return true;
        }

        queue<TreeNode*> levelQueue;
        levelQueue.push(root);
        while(!levelQueue.empty()){
            auto node = levelQueue.front();
            levelQueue.pop();

            if(!node->left && node->right){
                return false;
            }

            if(firstNull && (node->left || node->right)){
                return false;
            }

            if(!node->left || !node->right){
                firstNull = true;
            }
            if(node->left){
                levelQueue.push(node->left);
            }
            if(node->right){
                levelQueue.push(node->right);
            }
        }
        return true; 
    }
};
```
### 662. 二叉树最大宽度
https://leetcode-cn.com/problems/maximum-width-of-binary-tree/
层次遍历，对每个节点记录他的index值，他的子节点就是2*index && 2*idnex+1 。宽度就是每一层最后的那个节点的index减最开始的节点的index，再和记录的最大值作比较。这里最坑的就是用例可能会超出范围，直接用unsigned long long去给出回答。

```C++

class Solution {
public:
    typedef unsigned long long ull;
    ull maxWidth = 1;
    queue<pair<TreeNode*,ull>> levelQueue;
    int widthOfBinaryTree(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        levelQueue.push({root,1});
        while(!levelQueue.empty()){
            int size = levelQueue.size();
            if(size == 1){
                auto nodeP = levelQueue.front();
                levelQueue.pop();
                levelQueue.push({nodeP.first,1});
            }
            ull leftidx,rightidx;
            for(int i=0;i<size;i++){
                auto nodeP = levelQueue.front();
                levelQueue.pop();
                auto node = nodeP.first;
                if(i == 0){
                    leftidx = nodeP.second;
                }
                if(i == size-1){
                    rightidx = nodeP.second;
                }

                if(node->left){
                    levelQueue.push({node->left,(2*nodeP.second)});
                }
                if(node->right){
                    levelQueue.push({node->right,(2*nodeP.second+1)});
                }

            }
            maxWidth = max(maxWidth,(rightidx - leftidx + 1));
        }
        return maxWidth;
    }
};
```
### 515. 在每个树行中找最大值

https://leetcode-cn.com/problems/find-largest-value-in-each-tree-row/

层序遍历，每一层遍历完将最大值传入vector

```C++

class Solution {
public:
    queue<TreeNode*> levelQueue;
    vector<int> ans;
    vector<int> largestValues(TreeNode* root) {
        if(root == NULL){
            return ans;
        }
        levelQueue.push(root);
        while(!levelQueue.empty()){
            int size = levelQueue.size();
            int levelMax = INT_MIN;
            for(int i=0;i<size;i++){
                auto node = levelQueue.front();
                levelQueue.pop();
                levelMax = max(levelMax,node->val);
                if(node->left){
                    levelQueue.push(node->left);
                }
                if(node->right){
                    levelQueue.push(node->right);
                }
            }
            ans.push_back(levelMax);
        }
        return ans;
    }
};
```
### 919. 完全二叉树插入器
https://leetcode-cn.com/problems/complete-binary-tree-inserter/

每层节点入队列，判断如果该节点有左右子树，证明该节点不可能作为插入的父节点，从队列中删除。如果左右子树有一个没有，证明该节点就是插入的父节点，在队列中保留。所以队列中存储的是可以作为插入父节点的节点。插入的时候从队列头获取节点就可以了，如果它的左子树是空的，就插入；如果右子树是空的，插入之后它就没有位置给其他节点插入了，再pop出队列即可。

```C++
class CBTInserter {
public:
    CBTInserter(TreeNode* root) {
        this->root = root;
        parentsQueue.push(root);
        while(!parentsQueue.empty()){
            auto node = parentsQueue.front();
            if(node->left){
                parentsQueue.push(node->left);
            }else{
                break;
            }
            if(node->right){
                parentsQueue.push(node->right);
            }else{
                break;
            }
            parentsQueue.pop();
        }
    }
    
    int insert(int v) {
        auto node = parentsQueue.front();
        if(!node->left){
            TreeNode* left = new TreeNode(v);
            node->left = left;
            parentsQueue.push(node->left);
            return node->val;
        }else{
            TreeNode* right = new TreeNode(v);
            node->right = right;
            parentsQueue.push(node->right);
            parentsQueue.pop();
            return node->val;
        }
    }
    
    TreeNode* get_root() {
        return this->root;
    }

private:
    TreeNode* root;
    queue<TreeNode*> parentsQueue;
};
```
  
## 特殊遍历