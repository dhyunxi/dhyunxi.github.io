


- 二叉搜索树相对一般的树而言进行了排序，从而方便了查找


# 定义
---
- 对于每一个结点，其值大于所有左子树结点，小于所有右结点值
- 左右子树都是二叉搜索树


# 操作
---
## 查找
- 判断根结点大小
	- 等于：输出
	- 小于：进入右子树
	- 大于：进入左子树
```c++
//递归算法
Position find(BST T,ElemType X)
{
	if(T->data==X) return T.position;
	else if(T->data<X) return find(T->right,X);
	else return find(T->left,X);
}

//尾递归用循环代替
Position find(BST T,ElemType X)
{
	while(T){
		if(T->data==X){
			return T.position;
		}else if(T->data<X){
			T=T->right;
		}else{
			T=T->left;
		}		
	}
	return NotFind;
}
```
- 插入的效率与树高正相关

## 插入
- 若指针为空，插入。否则：
	- 若该结点值小：进入右子树
	- 大：进入左子树

## 删除
- 关键在于删除后子树的处理
- 若删除的是叶子：直接删除，将父链接置空
- 若只有一个子节点： 顺位上链
- 若有两个子节点：
	- 取右子树中最小元素替代
	- 取左子树中最大元素替代
```c++
BinarySearchTree Delete(BinarySearchTree BST,ElemType X)
{
	Position tem;
	if(!BST) return NotFind;
	else if(X<BST->data)
		BST->left=Delete(BST->left,X);
	else if(X>BST->data)
		BST->right=Delete(BST->right,X);
	else{
		if(BST->left&&BST->right){
			tem=FindMin(BST->right);
			BST->data=tem->data;
			BST->right=Delete(BST->right,tem);
		}else{
			tem=BST;
			if(!BST->left){   //左子树空
				BST=BST->right;
			}else{
				BST=BST->left;
			}
			delete tem;
		}
	}
	return BST;
}
```


## 判断是否为二叉搜索树
- 递归：
	- 判断左子树是否都小于(等于)根
	- 判断右子树是否都大于(等于)根
	- 判断左右子树是否是二叉搜索树
```c++
bool IsBST(BinarySearchTree T)
{
	if(T==NULL) return 1;
	if(FindMin(T->right)<T->data)
		return 0;
	else if(FindMax(T->left)<=T-data)
		return 0;
	else{
		return IsBST(T->left)&&IsBST(T->right);
	}
}
```

## 判断不同输入序列的二叉搜索树是否相同
- 建两棵树，遍历比较
- 不建树：
	- 比较第一个数字是否相同
		- 不相同则不一样
		- 相同则以第一个数字为基准，将后续数字分成小的和大的
		- 重复比较子序列
- 建一棵树，用其他的序列比较
	- 用flag域指示结点是否被访问过
```c++
//输入一串不含0序列，以此为基准建树
//输入一串不含0序列，以0为结尾，判断这串序列对应的树是否相同
typedef struct BSTNode{
	int data;
	int flag=0;
	BSTNode *left,*right;
}node,*BinarySearchTree;

//在T中匹配x，若匹配成功，将flag设置为1，返回True，否则返回0
//循环版本
bool judge(BinarySearchTree T,int x){
	if(T==NULL) return 0;
	while(T->flag){
		if(x<=T->data){
			T=T->left;
		}else{
			T=T->right;
		}
	}
	if(T->data==x){
		T->flag=1;
		return 1;
	}
}
//递归版本
bool judge(BinarySearchTree T,int x)
{
	if(T==NULL) return 0;
	if(T->flag){   //树非空，结点被匹配过
		if(x<=T->data) return judge(T->left,x);
		else return judge(T->right,x);
	}else{    //新的结点
		if(T->data==x) {
			T->flag=1;
			return 1;
		}
		else return 0;
	}
}


bool IsSame(BinarySearchTree T,int N){   //T是基树，N是总结点树
	int x; 
	int n=0;
	cin>>x;
	while(x){
		n++;
		if(!judge(T,x)){
			return 0;
		}
		cin>>x;
	}
	if(n==N) return 1;
	else return 0;
}

void ResetFlag(BinarySearchTree T)
{
	if(T==NULL) return ;
	else{
		T->flag=0;
		ResetFlag(T->left);
		ResetFlag(T->right);
	}
}
```


# 平衡二叉树
---
- ASL：平均查找长度
- 结点的查找长度等于深度+1
- ASL与树的深度有关
- 为了让ASL最小，我们希望树的深度最小

## 概念
- 平衡因子(Balance Factor, BF)：结点的平衡因子=左子树高度-右子树高度
- 平衡二叉树(AVL树)：每个结点的平衡因子绝对值不超过1

## 性质
- 设高度h的AVL树最少结点树为$n_{h}$，则有$n_{h}=n_{h-1}+n_{h-1}+1$
- $n_{h}$是指数阶的
- AVL树近乎完全：度一结点被限制在下面


## 插入
- 若按照二叉搜索的方式插入，容易出现不平衡
- 利用旋转来调整
	- RR插入：右单旋
				![[RR单旋]]
	- LL插入：左单旋
	- RL插入：考虑相邻的三个结点
				![[RL旋转]]
		- X插在CL或CR上无影响


# B-树
---
- 将平衡的概念拓展到多叉树，呈现多路的特性
- 适用于磁盘管理

## 性质
- m叉的B-树有如下结构
	- 每个结点储存有0~m-1个关键字，关键字有序排列
	- 在首尾和相邻关键字间加入指针域，指向子结点，每个关键字有左右子结点
- m叉的B-树有如下性质
	- 树中的每个结点至多有m棵子树
	- 根结点要么没有子树，要么有至少两棵子树
	- 设定叶子结点为失败结点，标志查找失败；称失败结点的上一层为终端结点
	- 所有叶子结点出现在同一层
	- 除根之外的所有非终端结点有至少$\left\lceil  \frac{m}{2}  \right\rceil$棵子树