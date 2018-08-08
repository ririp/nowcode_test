# 牛客网剑指offer编程题库笔记
##### 题库一刷之后进行二刷总结，整理顺序按考点来。
## 1   包含min函数的栈
#### 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
###### 用两个栈实现，每次push操作，将数据压入stk1，并将数据与stk2的栈顶比较，若小于stk2栈顶的值，则压入当前元素，该操作保证，stk2的栈顶一直存储着压入数据的最小值。而每次pop操作，比较stk1栈顶元素，也就是当前元素是否是stk2的栈顶元素，如果是，则stk2和stk1都需要将该元素pop出去。

```
class Solution {
private:stack<int>stk1,stk2;//stk1存正常data，压入过程中每次与stk2栈顶比较，将更小的压入，这样stk2栈顶就是最小值
public:
    void push(int value) {
        stk1.push(value);
        if(stk2.empty())stk2.push(value);
        if(value<=stk2.top())stk2.push(value);//注意这里等于也要压入，不然当有重复的最小值时，会被pop掉
    }
    void pop() {
        if(stk1.top()==stk2.top())stk2.pop();//如果pop出栈的数恰是最小值，那么stk2也得pop该最小值，因为此时data中已经没它了
        stk1.pop();
    }
    int top() {
        return stk1.top();
    }
    int min() {
        return stk2.top();
    }
};
```

## 2  二叉树中和为某一值的路径
#### 输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)。
###### 看到路径的定义（树的根节点到叶节点），想到深度遍历的递归实现。定义一个容器tmp存储单个可能路径，然后用容器res存储所有可能的路径。每一次递归，先将当前节点值从tmp容器尾部插入，然后判断当前节点是不是叶子节点且是否等于sum，若是则表示遍历完一条路径且符合，则将tmp的结果存到res容器，若否则继续递归左右子树，并将要求的值改为sum-root->val（已经加入temp的值），需要注意dfsfind函数最后的tmp.pop_back()操作，tmp会保存每一条路径，当不符合条件就不会存进res，每个tmp.push_back(root->val)对应一个tmp.pop_back()，当开始下一条路径的查找时temp为空。
```
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    void dfsfind(TreeNode* root,int sum){
        if(root==NULL)return;//递归实现需要注意跳出递归的条件
        tmp.push_back(root->val);
        if(root->left==NULL&&root->right==NULL&&root->val==sum)//到最底的叶子节点才算路径
        res.push_back(tmp);//当递归到最底，才将tmp push进res
        else{
            if(root->left)
                dfsfind(root->left,sum-root->val);
            if(root->right)
                dfsfind(root->right,sum-root->val);
        }
        tmp.pop_back();//只有符合条件的才会push进res，不然深度遍历完tmp往上返回，把拿进的都pop
    }
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
        if(expectNumber==0)return res;
        dfsfind(root,expectNumber);
        return res;
    }
};
```
## 3  从上往下打印二叉树
#### 从上往下打印出二叉树的每个节点，同层节点从左至右打印。
###### 层次遍历，最简单的是用队列实现。pop节点的同时push进左右子树。
```     
      9
    /   \
   5     15
  / \   /  \        
 2   3 12   18
```
###### 对于如上所示的二叉树，先把根节点9 push进队列，然后返回队列队头（此时结果是9），pop队头，将队头的左右子树5和15依次push进队列，继续做循环，pop队头的5（此时结果是9 5），push进5的左右子树，依次为2 3，此时队列中队头到队尾依次是15 2 3，循环操作……
```
/*
struct TreeNode {
	int val;
	struct TreeNode *left;
	struct TreeNode *right;
	TreeNode(int x) :
			val(x), left(NULL), right(NULL) {
	}
};*/
class Solution {
public:
    vector<int> PrintFromTopToBottom(TreeNode* root) {
    queue<TreeNode*>q;
    vector<int>result;
    if(!root)return result;//不加会溢出
    q.push(root);
    TreeNode* node;
        while(!q.empty()){
        node=q.front();
        result.push_back(node->val);
        q.pop();
        if(node->left)q.push(node->left);
        if(node->right)q.push(node->right);
        }
    return result;
    }
};
```
## 4 二叉搜索树的后续遍历序列
#### 输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
###### 二叉搜索树/二叉查找树Binary Search Tree  它或者是一棵空树，或者是具有下列性质的二叉树： 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值； 它的左、右子树也分别为二叉排序树。<br>BST的后序遍历合法序列是，对于一个序列S，最后一个元素是x （也就是根），如果去掉最后一个元素的序列为T，那么T满足：T可以分成两段，前一段（左子树）小于x，后一段（右子树）大于x，且这两段（子树）都是合法的后序序列，对两个子树序列递归判断。
```
class Solution {
public:
    bool judge(vector<int> temp,int lef,int rig){//temp是原序列，lef，rig表示此次判断起始位置和结束位置
        if(lef>=rig)return true;//l==r对应的是叶子结点，l>r对应的是空树，这两种情况都是合法的二叉搜索树
        int i=rig;
        while(i>lef && temp[i-1]>temp[rig])i--;//从根节点开始，找出右子树序列的初始位置
        for(int j=i-1;j>=lef;j--)if(temp[j]>temp[rig])return false;//如果左子树序列有大于根节点的，则表示不是合法序列
        return judge(temp,lef,i-1) && judge(temp,i,rig-1);
    }
    
    bool VerifySquenceOfBST(vector<int> sequence) {
      if(sequence.size()==0)return false;
      return judge(sequence,0,sequence.size()-1);
    }
};

```
## 5 栈的压入、弹出序列
#### 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
###### 直接用一个临时栈temp来模拟输入输出。把pushV的序列按顺序压入temp，压入过程中，将temp的栈顶（也就是下一个要pop出去的数）和popV序列作对比，若相等，则把temp的栈顶pop，直到不相等为止。如果昨晚整个操作，temp栈为空则返回true，否则说明压入弹出顺序不匹配。
```
class Solution {
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        int n=pushV.size();
        stack<int>temp;
        int index=0;
        for(int i=0;i<n;i++){
            temp.push(pushV[i]);//压入临时栈
        while(popV[index]==temp.top()&&index<n){//如果出栈等于临时栈的栈顶，将栈顶pop，弹出序列index+1
            temp.pop();
            index++;//这边只有栈顶等于当前弹出序列数值才需要后移，所以不能用for
        }
        }
        if(temp.empty())return true;
        else return false;
    }
};
```
