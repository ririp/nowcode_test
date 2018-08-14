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
## 6 数值的整数次方
#### 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
###### 快速幂。假设要求2的11次方，11的二进制为1011，所以11 = 2³×1 + 2²×0 + 2¹×1 + 2º×1，所以2的11次方可表示为2的8次方，2的2次方，2的1次方的乘积。以上述求2的11次方为例，代码流程如下：先对exponent取绝对值得到a，用a&1判断数a的二进制最后一位为不为1，11的二进制1011与1（0001）与运算，当a的二进制最后一位为1时，result=reslut*2=2，base=2*2=4，a右移为101；继续循环result=2*4=8……；快速幂的关键在于将指数用二进制表示，再将底相乘。
###### &运算，两位同时为1才得1，不然为0，&与&&都表示当两遍的表达式都成立时，才返回为true，但是&&有短路的现象，使用&&如果第一个表达式是false，那么程序直接返回false，但是&不会。
```
class Solution {
public://快速幂，2的11次方，即2的1+2+8次方，此算法复杂度更小
    double Power(double base, int exponent) {
    if(exponent==0)return 1;
    double result=1.0;
    int a=abs(exponent);
    while(a){
        if(a&1)result*=base;
        base*=base;
        a>>=1;
    }
    return (exponent>0)?result:1/result;
    }
};
```
## 7 调整数组顺序使奇数位于偶数前面
#### 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
###### 常规操作，遍历数组将偶数找出存入临时队列，遍历完之后将队列里的偶数依次push进数组（从后面）。
```
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        queue<int>temp;
        vector<int>::iterator it;
        //it=array.begin();
        for(it=array.begin();it!=array.end();){
            if(*it%2==0){
                temp.push(*it);
                it=array.erase(it);//erase后返回删除元素的下一个迭代器，相当于做了it++的操作
            }
            else it++;
        }
        while(!temp.empty()){
            array.push_back(temp.front());
            temp.pop();
        }
    }
};
```
## 8 链表中倒数第k个结点
#### 输入一个链表，输出该链表中倒数第k个结点。
###### 因为不遍历没法获取链表长度，所以用快慢（先走后走）指针，快指针先走k-1步，到达第k个节点，慢指针开始走，当先走的快指针到达时，慢指针到达倒数第k个节点
```
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* FindKthToTail(ListNode* pListHead, unsigned int k) {
    if(k<=0||pListHead==NULL)return NULL;
    ListNode* prior=pListHead;
    ListNode* later=pListHead;
    for(int i=1;i<k;i++){
        if(prior->next!=NULL)prior=prior->next;
        else return NULL;//k大于链表长度
    }
    while(prior->next!=NULL){
        prior=prior->next;
        later=later->next;
    }
    return later;
    }
};
```
## 合并两个排序的链表
#### 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
###### 两个有序链表合成，就不需要重排了，这里用非递归实现，新建newhead和cur，cur记录当前要插的节点位置，判断两个链表头结点存储值的大小，将更小的连接到cur的后面，然后cur后移一格，执行完while循环若两链表有剩余，将剩余部分街上，最后返回新的头结点，即newhead->next；
```
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        if(!pHead1&&!pHead2)return NULL;
        if(!pHead1)return pHead2;
        if(!pHead2)return pHead1;
        ListNode* cur=new ListNode(5);//随意初始化值,但一定要初始化，不能为NULL，因为listnode是一个指定类，且需要赋值
        ListNode* newhead=cur;
        while(pHead1&&pHead2){
            if(pHead1->val < pHead2->val){
                cur->next=pHead1;
                pHead1=pHead1->next;
            }
            else{
                cur->next=pHead2;
                pHead2=pHead2->next;
            }
            cur=cur->next;
        }
        if(pHead1)cur->next=pHead1;//如果链表1还没完，直接把链表一剩下的接上去
        if(pHead2)cur->next=pHead2;
        return newhead->next;
    }
};
```
## 反转链表
#### 输入一个链表，反转链表后，输出新链表的表头。
###### 用三个指针完成，过程简述如下：
```
对于链表2->3->1->4->9,p指向头节点2，q指向下一个节点3，先断开头结点，此时2（p） 3（q）->1->4->9
做第一次循环
r=q->next      2（p）    3（q）->1（r）->4->9
q->next=p      2（p）<-3（q）    1（r）->4->9
p=q            2  <-3（p,q）     1（r）->4->9
q=r            2  <-3（p）       1（q）->4->9
下一次循环后    2  <-3 <-1（p）       4（q）->9
重复上述过程 p就是新的头结点
```
```
class Solution {
public:
    ListNode* ReverseList(ListNode* pHead) {
     if(pHead==NULL)return NULL;
     ListNode* p = pHead;
     ListNode* q = pHead->next;
     ListNode* r;
     pHead->next=NULL;
     while(q){
         r=q->next;
         q->next=p;
         p=q;
         q=r;
     }
     return p;
    }
};
```
## 树的子结构
#### 输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）
###### 题目中pRoot1指向A的根，pRoot2指向B的根。需要熟悉递归思想，hassubtree用来判断B是不是A从根节点开始的子结构，如果不是分别从A的左右子树继续找；如果二叉树A的根节点与B的根节点相同，调用judge函数。judge函数也是递归实现。具体见代码注释。
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
    bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2)
    {
     bool result=false;//默认结果
     if(pRoot1!=NULL&&pRoot2!=NULL){
         if(pRoot1->val==pRoot2->val){//如果二叉树A的根节点与B的根节点相同
             result=judge(pRoot1,pRoot2);//开始判断B是不是A从根节点开始的子结构
         }
         if(!result){//没找到，继续从A的左子树开始找包不包含B
             result=HasSubtree(pRoot1->left,pRoot2);
         }
         if(!result){//没找到，继续从A的右子树开始找包不包含B
             result=HasSubtree(pRoot1->right,pRoot2);
         }
     }
        return result;
    }
    bool judge(TreeNode* pRoot1, TreeNode* pRoot2){
        if(pRoot2==NULL)return true;//B树遍历完了，都能对上，返回true
        if(pRoot1==NULL&&pRoot2!=NULL)return false;//A树结束了，B树还有，则不为子结构，
        if(pRoot1->val==pRoot2->val)//如果相同就继续判断 各自的左右是否相同
            return judge(pRoot1->left,pRoot2->left)&&judge(pRoot1->right,pRoot2->right);//不想同返回false
        else return false;//若当前节点不同就返回false
    }
};
```
## 二进制中1的个数
#### 输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。
```
class Solution {
public:
     int  NumberOf1(int n) {
     int comp=1,count=0;
     for(int i=0;i<32;i++){//一个int 4个字节一共32位
         if((comp&n)!=0)count++;//&符号直接将n表示成二进制，然后与comp与，当该位为1时输出不为0，因为是二进制结果，所以不能==1
         comp=comp<<1;//comp左移，即从后往前比较，即第i+1位为1，其他均为0
     }
     return count;
     }
};
//另一种最优解是，n=n&（n-1），该操作会消去一个二进制的1，count++，当n为0时，count即是结果
```
## 二叉搜索树与双向链表
#### 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。
###### 用两结点分别指向链表的头尾。具体做法在代码中注释。
```
例：
     10
    /   \
   7     15
  / \   /  \        
 2   9 12   18
 先递归到最左端结点2，此时还会执行一次递归（第一个if）返回到结点2，执行第二个if，head和tail都指向结点2
 返回头结点head后回到结点7，执行else语句，结点7的左指针指向tail即2，然后tail右指针指向结点7，尾巴移到结点7，此时2<->7，
 然后递归语句到结点9重复上述操作。
```
```
class Solution {

public:
    TreeNode* head=NULL;//已完成双向链表的头结点
    TreeNode* tail=NULL;//已完成双向链表的尾结点
    TreeNode* Convert(TreeNode* pRootOfTree)
    {
        if(pRootOfTree==NULL)return NULL;//用来判断到达最底端，返回null仅仅表示递归的终点
        Convert(pRootOfTree->left);//向左遍历
        if(head==NULL){//当前值为null时的上一层，也就是最左端执行
            head=pRootOfTree;//最初的双向链表只有当前点，所以头尾都是它
            tail=pRootOfTree;
        }
        else{
            pRootOfTree->left=tail;//第二个节点开始，左边连接链表的尾
            tail->right=pRootOfTree;//链表尾右边连接当前节点
            tail=pRootOfTree;//链表尾变为当前节点
        }
        Convert(pRootOfTree->right);//向右遍历
        return head;//返回头节点
    }
};
```
## 复杂链表的复制
#### 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）
###### 分三步操作，<br>1.先复制每个结点，1->2->3->4 =》1->1->2->2->3->3->4->4 得到新链表2<br>2.然后复制特殊指针；如果结点1的特殊指针指向3，那么第一步操作完后，对于新链表2 结点3后面也是3，该结点就是新链表结点1的特殊指针位置，所以我们可以用A1.random = A.random.next描述特殊指针（A1是A的复制结点）<br>3.拆分链表
```
/*
struct RandomListNode {
    int label;
    struct RandomListNode *next, *random;
    RandomListNode(int x) :
            label(x), next(NULL), random(NULL) {
    }
};
*/
class Solution {
public:
    RandomListNode* Clone(RandomListNode* pHead)
    {
        if(pHead==NULL)return NULL;
        RandomListNode* cur=pHead;
        while(cur!=NULL){//复制每个结点，如复制结点A得到A1，将结点A1插到结点A后面；对于链表1->2->3->4
            RandomListNode* clonenode=new RandomListNode(0);//1->2->3->4 ;0clonenode
            clonenode->label=cur->label;//1->2->3->4 ;1clonenode
            RandomListNode* nextnode=cur->next;//1 cur->2 next->3->4;1clone
            cur->next=clonenode;//1 cur->1clone;2next->3->4
            clonenode->next=nextnode;//1 cur->1clone->2next->3->4
            cur=nextnode;//1 ->1clone->2cur->3->4
        }
        cur=pHead;//循环完重新回头
        while(cur!=NULL){//重新遍历链表，复制老结点的随机指针给新结点，如A1.random = A.random.next;
            cur->next->random=cur->random==NULL?NULL:cur->random->next;
            cur=cur->next->next;
        }
        cur=pHead;
        RandomListNode* clonehead=pHead->next;
        while(cur!=NULL){//拆分链表，将链表拆分为原链表和复制后的链表
            RandomListNode* clonenode=cur->next;//1cur 1clonehead/clonenode 2 2 3 3 4 4
            cur->next=clonenode->next;//1cur->2   分主链
            clonenode->next= clonenode->next==NULL?NULL:clonenode->next->next;//分克隆链
            cur=cur->next;//主链后移
        }
        return clonehead;
    }
};
```
## 字符串的排列
#### 输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。<br>输入描述:<br>输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。
###### fun函数执行流程如下图（图片源自牛客网~），pos记录字符串执行下标。对于重复值，利用set中值不重复的特性筛选，然后放回容器。
![image](https://github.com/ririp/nowcode_test/blob/master/image/8.14github.png)
```
class Solution {
public://回溯法
    set<string> res;//set里面的值不重复
    void fun(string str, int pos)
    {
        if (pos == str.size())//此时str已经是一次完整的全排列，如bca
        {
            res.insert(str);
            return;
        }
        for (int i = pos; i < str.size(); i++)
        {
            swap(str[i], str[pos]);
            fun(str, pos + 1);
        }
    }
    vector<string> Permutation(string str) {
        res.clear();
        vector<string> st;//用vector重新放set得到的结果
        if (str.size() == 0)return st;
        fun(str, 0);//从位置0开始
        set<string>::iterator it;
        for (it = res.begin(); it != res.end(); it++)
            st.push_back(*it);
        return st;
    }
};
```
## 求1+2+3+...+n
#### 求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。
###### 用&&短路特性，当&&左边为false时，将不再计算右边的表达式。
```
总结:
&&与&的区别为,&无论左边的结果是否为真,都将继续运算右边的逻辑表达式;
&&左边的值为false时,将不会继续运算其右边的逻辑表达式,结果为false;
||与|的区别为,|无论其左边的结果是否为true,都将继续运算右边的逻辑表达式;
||当计算完左边的逻辑表达式且其结果为true时,将不会继续计算右边的逻辑表达式,结果为true;
短路与&&和短路||都提高了逻辑表达式的计算效率.
```

```
class Solution {
public:
    int Sum_Solution(int n) {
        int count=n;
        count&&(count +=Sum_Solution(n-1));
        return count;
    }
};
```
## 不用加减乘除做加法
#### 写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。
##### 不能用加减乘除，只能用 或 异或 与 运算了吧。对于二进制而言，两个数异或：相当于每一位相加，而不考虑进位；两个数相与，并左移一位，相当于求得进位。两数相加直到进位为0时，即可直接返回异或值（无进位），流程如下：
```
对于12（num1)和9（num2）,二进制分别为1100和1001
1 temp=1100^1001=0101                    
  num2=(11001&1001)<<1=1000<<1=10000    16  
  num1=0101                             5
  
2 temp=0101^10000=10101                 
  num2=(0101&10000)<<1=00000=0          0
  num1=10101                            21
  返回结果 21
```
```
class Solution {
public:
    int Add(int num1, int num2)
    {
        while(num2!=0){
            int temp=num1^num2;
            num2=(num1&num2)<<1;
            num1=temp;
        }
        return num1;
    }
};
```
## 9 跳台阶
#### 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。
###### 经典问题。以下摘自网络。
###### 首先我们考虑最简单的情况。如果只有1级台阶，那显然只有一种跳法。如果有2级台阶，那就有两种跳的方法了：一种是分两次跳，每次跳1级；另外一种就是一次跳2级。现在我们再来讨论一般情况。我们把n级台阶时的跳法看成是n的函数，记为f(n)。当n>2时，第一次跳的时候就有两种不同的选择：一是第一次只跳1级，此时跳法数目等于后面剩下的n-1级台阶的跳法数目，即为f(n-1)；另外一种选择是第一次跳2级，此时跳法数目等于后面剩下的n-2级台阶的跳法数目，即为f(n-2)。因此n级台阶时的不同跳法的总数f(n)=f(n-1)+(f-2)。<br>我们把上面的分析用一个公式总结如下：
```
          /  1                          n=1
f(n)=        2                          n=2
          \  f(n-1)+(f-2)               n>2

分析到这里，相信很多人都能看出这就是我们熟悉的Fibonacci序列。
```

```
class Solution {
public:
    int jumpFloor(int number) {//斐波那契数列，和放矩阵类似
        if(number<=2)return number;
        int one=1,two=2,next=0;
        for(int i=1;i<number-1;i++){//相当于先生成前两个数1，2，下一个数为前两个数相加的和，one表示前两个数，two表示前一个数，next表示当前的数
            next=one+two;
            one=two;
            two=next;
        }
        return next;
    }
};
```
## 10 变态跳台阶
#### 一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
###### 分析：用Fib(n)表示青蛙跳上n阶台阶的跳法数，青蛙一次性跳上n阶台阶的跳法数1(n阶跳)，设定Fib(0) = 1；<br>当n = 1 时， 只有一种跳法，即1阶跳：Fib(1) = 1;<br>当n = 2 时， 有两种跳的方式，一阶跳和二阶跳：Fib(2) = Fib(1) + Fib(0) = 2;<br>当n = 3 时，有三种跳的方式，第一次跳出一阶后，后面还有Fib(3-1)中跳法； 第一次跳出二阶后，后面还有Fib(3-2)中跳法；第一次跳出三阶后，后面还有Fib(3-3)中跳法<br>Fib(3) = Fib(2) + Fib(1)+Fib(0)=4;<br>当n = n 时，共有n种跳的方式，第一次跳出一阶后，后面还有Fib(n-1)中跳法； 第一次跳出二阶后，后面还有Fib(n-2)中跳法..........................第一次跳出n阶后，后面还有 Fib(n-n)中跳法.<br>Fib(n) = Fib(n-1)+Fib(n-2)+Fib(n-3)+..........+Fib(n-n)=Fib(0)+Fib(1)+Fib(2)+.......+Fib(n-1)<br>又因为Fib(n-1)=Fib(0)+Fib(1)+Fib(2)+.......+Fib(n-2)<br>两式相减得：Fib(n)-Fib(n-1)=Fib(n-1)         =====》  Fib(n) = 2*Fib(n-1)     n >= 2<br>代码中的注释给出了另外一种比较好理解的思路
```
class Solution {
public:
    int jumpFloorII(int number) {//相当于n个数的子集，而最后一个台阶必须跳，所以是2的n-1阶层，理解为n个台阶，在每个台阶跳或者不跳，两个选择。
    if(number==1)return 1;
    return 2*jumpFloorII(number-1);//f(n)=f(n-1)+f(n-2)...f(0);f（n-1）=f（n-2）+……+f（0）；所以f(n)=2*f(n-1)
    }
};
```

## 11 矩阵中的路径
#### 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。
```
对于题目中的样例
a b c e 
s f c s 
a d e e
```
```

class Solution {
public:
    bool hasPath(char* matrix, int rows, int cols, char* str)
    {
       if(matrix==NULL||rows<1||cols<1||str==NULL) return false;
       bool *flag=new bool[rows*cols];//记录是否走过
       memset(flag,false,rows*cols);//都先初始化为false，表示未走过
       for(int i=0;i<rows;i++)//两个for循环表示从位置i，j开始走
       {
           for(int j=0;j<cols;j++)
           {
               if(judge(matrix,rows,cols,i, j,str,0,flag))
               {
                   return true;
               }
           }
       }
        delete[] flag;
        return false;
    }
     /*参数说明：
     matrix是矩阵，rows是矩阵的行数，cols是矩阵列数，i，j是当前位置的行坐标和列坐标，
     str是要匹配的字符串数组，k表示匹配字符串的第k个值，flag是表示矩阵某位置是否被访问的数组*/
     
    bool judge(char* matrix,int rows,int cols,int i,int j,char* str,int k,bool* flag)
    {
        //因为是一维数组存放二维的值，index值就是相当于二维数组的（i，j）在一维数组的下标
       int index = i * cols + j;
        //flag[index]==true,说明被访问过了，那么也返回false;
       if(i<0 || i>=rows || j<0 || j>=cols || matrix[index]!=str[k] || flag[index]==true)
               return false;
        //字符串已经查找结束，说明找到该路径了
       if(str[k+1]=='\0') return true;
        //向四个方向进行递归查找,向左，向右，向上，向下查找
       flag[index] = true;//标记访问过
       if(  judge(matrix, rows, cols, i - 1, j,     str, k + 1, flag)
          ||judge(matrix, rows, cols, i + 1, j,     str, k + 1, flag)
          ||judge(matrix, rows, cols, i,     j - 1, str, k + 1, flag)
          ||judge(matrix, rows, cols, i,     j + 1, str, k + 1, flag))
        {
            return true;
        }
        flag[index] = false;//如果四个方向的路都没走通，则说明当前位置不需要走，重新置为flase；
        return false;
    }
};
```

## 12 机器人的运动范围
#### 地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？
###### 和上题类似，对于这类问题，即使是编程题，也推荐分开写函数，一步步解决问题，有助于理清思路。先解决数位之和，然后实现四方向走的递归，最后主函数调用从初始点开始走。
```
class Solution {
public:
    int sum(int a){//获取数位之和
        int sum=0;
        while(a){
            sum+=a%10;
            a/=10;
        }
        return sum;
    }
    int moving(int threshold,int i,int j,int rows,int cols,bool* flag){
        int count=0;
        if(i>=0&&i<rows&&j>=0&&j<cols&&sum(i)+sum(j)<=threshold&&threshold&&!flag[i*cols+j]){//注意要将条件考虑周全
            flag[i*cols+j]=true;
            count=1+moving(threshold,i,j+1,rows,cols,flag)
                   +moving(threshold,i+1,j,rows,cols,flag)
                   +moving(threshold,i-1,j,rows,cols,flag)
                   +moving(threshold,i,j-1,rows,cols,flag);
        }
        return count;
    }
    int movingCount(int threshold, int rows, int cols)
    {
        bool *flag =new bool[rows*cols];
        memset(flag,false,rows*cols);//flag初始化为false
        int count=moving(threshold,0,0,rows,cols,flag);//表示从0，0开始走
        delete []flag;
        return count;      
    }
};
```
## 13 连续子数组的最大和
#### HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)
###### 贪心算法，maxsum记录目前sum的最大值，sum记录连续子向量的和，当sum<0时说明之前的子向量和为负数（此时maxsum记录了之前那段子向量中连续和的最大值），则将sum置为0，继续计算之后的连续子向量和。
```
class Solution {
public:
    int FindGreatestSumOfSubArray(vector<int> array) {
    int sum=array[0];
    int maxsum=array[0];
    for(int i=1;i<array.size();i++){
        if(sum<0)sum =0;
        sum+=array[i];
        maxsum=(sum>maxsum)?sum:maxsum;
    }
    return maxsum;
    }
};
```
