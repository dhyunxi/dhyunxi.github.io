
# 定义
---
- 树是n个节点的无圈联通图
	- 有且仅有一个根节点
	- 除根节点外，其余节点可分为若干子树

## 术语
- 结点
- 结点的度：结点拥有的子树数目
- 树的度：结点度的最大值
- 叶子结点、终端结点：没有子树的结点
- 内部节点：非终端结点
- 双亲
- 孩子
- 祖先
- 子孙
- 兄弟
- 层次
- 深度：到根结点的最少边数，根的深度为0
- 高度：到叶子结点的最大边数
- 树的深度、高度
- 有序树、无序树
- 森林：若干棵互不相交的树的集合

## 性质
- 边数等于结点数-1


## 操作
- 构造
- 析构
- 清空
- 判空
- 深度
- 高度
- 根
- 查询
- 赋值
- parent
- child
- sibling
- 插入
- 删除
- 遍历

# 二叉树
---
- 至多两个子结点的树
- 子树有左右之分
- 第i层至多$2^i$个结点
- 设叶子节点数和2度结点数为$n_{0},n_{2}$，则有$n_{0}=n_{2}+1$
	- Proof：记总结点数为$n$，1度结点数$n_{1}$，则
$$
\begin{aligned}
n=n_{0}+n_{1}+n_{2} \\
n=n_{1}+2n_{2}+1
\end{aligned}
$$
- n个结点的二叉树有n+1个空链域

## 特殊二叉树
- 满二叉树：每一层都是满的
- 完全二叉树：除最底层外，每一层都是满的；且非满层的结点按照从左向右填入
	- 特点：
		- 叶子结点只可能在最后两层出现
		- 左子树深度=右子树深度或右子树深度+1
		- 具有n个结点的完全二叉树深度为$\lfloor \log_{2}n \rfloor+1$
		- 按从上到下，从左到右编号，由1起始，则若某结点编号n
			- 左子结点编号$2n$
			- 右子结点编号$2n+1$
			- parent编号$\left\lfloor  \frac{n}{2}  \right\rfloor$
- 二叉搜索树
- 平衡二叉树
- 红黑树
- B树



## 存储结构
---
### 顺序存储结构
- 利用完全二叉树的编号特点，可以利用数组储存完全二叉树

### 链式存储结构
- 结点包含值域、左指针域、右指针域，有时添加父结点域
```c++
typedef struct BinaryTreeNode{
	DataType data;
	BinaryTreeNode *left,*right;
} BTNode;
```
- 链表头指向二叉树根结点

### 静态链表
- 构造结构数组，其中每个元素有三个域，左指针域，值域，右指针域
- 指针利用序号代替，-1代表NULL

## 遍历
- 前序遍历、中序遍历、后序遍历、层序遍历
### 深度优先---前中后序遍历
#### 算法
- 采用递归算法，以中序遍历为例
```c++
void TraverseBT(BTNode* root){
	if(root){
		TraverseBT(root->left);
		cout<<root->data;
		TraverseBT(root->right);
	}
}
```
- 利用栈模拟递归结构，以中序遍历为例
```c++
//先遍历左子树
//当左子树遇到NULL时返回上一层访问data，然后进入右子树
//当右子树遇到NULL时返回上一层，再返回上一层，然后进入右子树
//在进入右子树时将结点pop掉来实现连续返回两层
//可以发现只要遇到NULL我们就进入S.top()的右子树
void TraverseBT(BTNode *root){
	stack S;
	BTNode* ptr=root;
	while(ptr||!S.empty()){
		if(ptr){
			S.push(ptr);
			ptr=ptr->left;
		}else{
			ptr=S.top();
			cout<<ptr->data;
			S.pop();
			ptr=ptr->right;
		}
	}
}
```

- 由二叉树的前序和中序序列，或由后序和中序序列均能唯一确定一棵二叉树
	- 根据定义，前序序列一定以根节点为首，中序序列中根结点将整个序列分为左子树的中序和右子树的中序，以此递归

#### 应用
- 创建二叉树的存储结构---二叉链表
	- 以先序遍历为例，以#表示空子树
```c++
void CreatBT(BTNode* &T){
	cin>>ch;
	if(ch=='#')
		T=NULL;
	else{
		T=new BTNode;
		CreatBT(T->left);
		CreatBT(T->right);
	}
}
```
- 复制二叉树
```c++
void CopyBT(BTNode* &newT,BTNode* T){
	if(T==NULL){
		newT=NULL;
	}else{
		newT=new BTNode;
		newT->data=T->data;
		CopyBT(newT->left,T->left);
		CopyBT(newT->right,T->tight);
	}
}
```
- 计算二叉树深度
- 统计结点个数

### 广度有限---层序遍历
- 利用队列实现
	- 从根结点开始入队
	- 访问队首结点，并令其非空左右子结点入队


## 线索二叉树
- 应用最多的是中序线索二叉树
### 定义
- 在遍历中，我们将二叉树线性化为了序列，每个结点存在一个前驱和一个后继，但在树结构中，这部分信息无法直接得到，希望找到一种方法存储
- 重新规定结点的左右指针域：
	- 若结点有左右子树，则不变
	- 若结点无左子树，则左指针域存放前驱
	- 若结点无右子树，则右指针域存放后继
	- 增加标志域来指示存放内容
```c++
struct node{
	DataType data;
	node* left;
	node* right;
	bool LTag=0,RTag=0;     //标志域：0指示子树，1指示前驱后继
};
```
- 这个过程称为**线索化**
- 指向前驱和后继的指针称为**线索**
- 线索化的树称为**线索二叉树**
- 为了方便，为线索二叉树添加一个头节点。头结点的左指针指向根结点，右指针指向遍历中最后一个结点。并令遍历中的第一个结点的左指针指向头结点，遍历中最后一个结点的右指针指向头结点。

### 线索化
- 线索化的过程就是在遍历中修改空结点的过程
- 以中序线索化为例
```c++
node* pre;     //pre作为全局变量跟踪上一个访问的结点
//结点递归线索化
void Threading(node* p){
	if(p){
		Threading(p->left);
		if(!p->left){
			p->LTag=1；
			p->left=pre;
		}else{
			p->LTag=0;
		}
		if(!pre->right){
			pre->RTag=1;
			pre->right=p;
		}else{
			pre->RTag=0;
		}
		pre=p;
		Threading(p->right);
	}
}
//插入头结点
void InThreading(node* &head,node* T){
	head=new node;    //初始化头结点
	head->LTag=0;     //头结点的左指针指向根节点T
	head->RTag=1;     //头结点的右线索指向遍历中最后一个结点
	head->right=head; //初始化右线索指向自己
	if(!T){           //如果树是空的，则左指针也指向自己
		head->left=head;
	}else{
		head->left=T;
		pre=head;
		Threading(T);
		head->right=pre;
	}
}
```

### 线索位置
- 中序：
	- 前驱：
		- 若有左链：前驱是左子树中最右下的结点
		- 若无左链：
			- 若是左孩子：祖先中第一个右孩子
			- 若是右孩子：parent
	- 后继：
		- 若有右链：右子树中最左下结点
		- 若无右链：
			- 若是左孩子：parent
			- 若是右孩子：祖先中第一个左孩子

### 利用线索二叉树遍历
- 以中序为例
```c++
void InTraverse(node* head){
	//head是头结点，其左链是根，右链是遍历中最后一个结点
	node* p=head->left;
	while(p!=T)
	{
		while(p->LTag==0) p=p->left;     //进入最左下结点
		cout<<p->data;                   //访问
		while(p->RTag==1){               //顺右线索访问后继
			p=p->right;
			cout<<p->data;
		}
		p=p->right;                      //没有右线索则进入右子树
	}
}
```




# 树和森林
---
## 树的存储结构
### 双亲表示法
- 结构数组
	- 数据域data
	- parent域：指示父结点位置
- 求根和父结点很方便，但求孩子需要遍历整棵树

### 孩子表示法
- 结构数组+链表
	- 数据域data
	- child域：一个头指针，指示一个孩子链表
- 求孩子很方便
- 可以将双亲表示法和孩子表示法结合

### 孩子兄弟法
- 又称二叉树表示法或二叉链表表示法，即以二叉链表作为存储结构
- 多重链表
	- 数据域data
	- 指针域：
		- firstchild：指向第一个孩子结点
		- nextsibling：指向兄弟链表
- 若想进入x的第i个孩子，只需先访问第1个孩子，再访问第i-1个兄弟
- 可以增设parent域

## 森林与二叉树的转换
- 在树的二叉链表表示中，根是没有兄弟的，因此根的nextsibling域空置
- 将根看作兄弟，选择一个根作为总根，便是森林的二叉链表


# 哈夫曼树
---
## 概念
- 路径：从一个结点到一个结点的分支
- 路径长度：路径上的分支数目
- 结点的路径长度：从根到此结点的路径长度，即结点深度
- 树的路径长度：从树根到所有叶子的路径长度之和
- 权：结点或边具有的权重
- 树的带权路径长度：WPL
- **哈夫曼树**：假设有m个权重，对于正整数n$(n\geq m)$存在有m个叶子结点的n阶二叉树，将m个权重分别赋给这m个叶子结点，则这些二叉树中带权路径长度最小的那个，称为最优二叉树，即哈夫曼树


## 特点
- 哈夫曼树一定是满二叉树，没有度一结点
- 总结点数n，叶子结点数m，$n=2m-1$

## 构造：
- 容易发现，哈夫曼树中，权重越大的叶子结点越靠近根
	- 根据给定的n个权，构造n个根，分配权重
	- 构造一个新的根，取原先出权最小的两个根，将其作为新根的左右孩子，删除那两个根并加入新构造的二叉树
	- 重复直到只含一颗树为止

### 实现
- 设有n个叶子，则总结点数2n-1，可以构造一维结构数组
- 数组大小2n，0号位不使用，叶子存放在1~n号中
```c++
typedef struct Huffman_Tree_Node{
	int weight=0;      //权值
	int paren=0t,left=0,right=0;    //指针域
}HTNode,*HuffmanTree;

void CreateHT(HuffmanTree &HT,int n,int w[])
{
	if(n<=1) return ;
	int m=2*n-1;
	HT=new HTNode[m+1];
	for(int i=1;i<=n;i++)
	{
		HT[i].wight=w[i-1];
	}//载入权重
	//连结、删除、插入，循环执行n-1次
	for(int i=n;i<=m;i++){
		Select(HT,i,s1,s2)      //在HT的前i项中查找权最低的两项，将下标返回给s1、s2
		HT[s1].parent=i+1;
		HT[s2].parent=i+1;
		HT[i+1].left=s1;
		HT[i+1].right=s2;
		HT[i+1].wight=HT[s1].wight+HT[s2].wight;
	}
}
```

## 哈夫曼编码

### 概念
- 前缀编码：若方案中，任一编码不是另一编码前缀，则称前缀编码
- 后缀编码：若方案中，任一编码不是另一编码后缀，则称后缀编码
- 哈夫曼编码：对于n叶哈夫曼树，对每个结点的左分支赋0，右分支赋1，将从根到叶子结点经历的01序列作为该叶子的编码
	- 哈夫曼编码是前缀编码
	- 哈夫曼编码是最优前缀编码

### 实现
- 从叶子结点出发回溯，左分支写0，右分支写1
- 由于回溯是逆序的，所以最后写入时应逆序写入
```c++
typedef char* HuffmanCode;   //动态数组存储哈夫曼编码表
HuffmanCode HC[n+1];
//为了方便，数组的0号单元不使用
//每个单元按编码长度动态分配大小

void LeafCode(HuffmanTree HT,char* NodeCode,int k)  //求第k个叶子结点的哈夫曼编码
{
	char code[n];       //临时存储空间
	int length=0;      
	int PNode=HT[k].parent;
	while(PNode){
		if(HT[PNode].left==k){
			code[length]=0;
		}else{
			code[length]=1;
		}
		length++;
		k=PNode;
		PNode=HT[PNode].parent;
	}
	if(lenght==0){                              //没东西
		NodeCode=NULL;
	}else{                                      //倒序存储
		NodeCode=new char[length+1];
		for(int i=0;i<length;i++){       
			NodeCode[i]=code[length-i-1];
		}
		NodeCode[length]='\0'
	}
}

void CreateHuffmanCode(HuffmanTree HT,HuffmanCode &HC,int n)
{
	
}

```







