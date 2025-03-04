

- 模式匹配算法


# 定义
---
- 串是一种特殊的线性表，数据元素被限制为字符
> 串(string)，或字符串，是由零个或多个字符组成的有限序列
> 一般记为 $s="a_{1}a_{2}\dots a_{n}"$ 
> 用双引号括起来的是字符串的**值**
> n是字符串的**长度**
> **空串**：含零个字符的串
> **子串**：串中任意连续的若干字符   *字串在主串中的位置以字串的第一个字符在主串中的位置表示*  
> **空格串**：由一个或多个空格组成的串


# ADT
---
- 数据对象：字符集
- 操作：
	- 创建
	- 析构
	- 判空
	- 长度
	- 复制
	- 清空
	- 比较
	- 连接
	- 子串
	- 查找
	- 替换
	- 插入
	- 删除

# 存储结构
---
## 顺序存储
- 串多采用顺序存储
- 由于需要动态分配空间，串的存储多在堆上
```c++
const int MAXSIZE=1000;
struct SString{
	char s[MAXSIZE];
	int length=0;
};
```

### 模式匹配算法
- 子串的定位通常称为模式匹配或串匹配
- 主要介绍BF算法和KMP算法
#### BF算法（Brute-force）
- 一种简单直观的算法
- 遍历主串以查找子串
```c++
//重载下标运算符
SString::char operator[](int index){
	return this->s[index];
}


int index_BF(SString a,SString b,int pos)  //a为主串，b为字串，pos为查找起始位置
{
	int i=pos,j=0;      //i指示主串下标，j指示字串下标
	while(i<a.length&&j<b.length)  //主串与子串没有达到末尾
	{
		if(a[i]==b[j]){
			i++;
			j++;
		}else{
			i=i-j+1;
			j=0;
		}
	}
	if(j==b.length) return i-j;
	return 0;
}
```
- 设主串长度n，字串长度m
- 则最差时间复杂度可达O(m * n)


#### KMP算法
- 在BF基础上，利用部分字串优化，达到时间复杂度O(m+n)
- 特点：主串指针无需回溯
- **思路**：若匹配进行到主串S的第i个位置和模式T的第j个位置时发生错误，那么此时
$$
S[i]\neq T[j] 
$$
$$
\text{“ }T_{1}T_{2}\dots T_{j-1}\text{ "}=\text{“ }S_{i-j+1}S_{i-j+2}\dots S_{i-1}\text{ "}
$$
		BF算法中，将T向右滑动一位，重新开始匹配
		但这样的算法没有用到已匹配的部分子串，思考能否利用匹配的子串优化？
		如果$T_{2}\neq T_{1}$那么向右滑动一位显然不够
		希望向右滑动尽量大
		假如T的前j-1元子串中，存在k，使首k-1元字串与尾k-1元字串相等，即
		$$
T_{1}T_{2}\dots T_{k-1}=T_{j-k}T_{j-k+1}..T_{j-1}
$$
		那么，取最大的那个k，将T滑动使$T_{k}$对齐$S_{i}$即可
		**是否可能忽略更小的滑动情形？**
		假设存在正整数p，使T右滑p后$S_{i}$前的项是对齐的，且$p<j-k$，那么有
		$$
T_{1}T_{2}\dots T_{j-1-p}=S_{i-j+1+p}S_{i-j+1+p+1}\dots S_{i-1}=T_{1+p}T_{2+p}\dots T_{j-1}
$$
		这说明k至少可取j-p，与假设矛盾
		对于j=0的情形，$S_{i}$没有匹配的必要了，因此取k=-1，并认为$T_{-1}$与任意值匹配
- k的值仅由j和T本身确定，因此可以根据T，建立$j\to k$的映射，存储在数组next中
```c++
for(i<S.length&&j<T.length){
	if(j==-1||S[i]==T[j]){   
		i++;
		j++;
	}else{
		j=next[j];
	}
}
```
- 将next的构造视为T对T的KMP匹配，则next序列有递归结构
	$$
next[0]=-1
$$
$$
next[j+1]=
\left\{  
\begin{aligned}
匹配成功，next[j]+1 \qquad T_{next[j]}=T_{j}\\
向右滑动，检查T_{next[next[j]]}=T_{j} \qquad T_{next[j]}\neq T_{j}


\end{aligned}
\right.
$$
```c++
void get_next(SStrint T,int next[])
{
	int i=0,j=-1;               //i指示主串T，j指示模式T
	next[0]=-1;
	while(i<T.length){
		if(j==-1||T[i]==T[j]){
			i++;
			j++;
			next[i]=j;
		}else{
			j=next[j];
		}
	}
}
```
- next构造时间复杂度O(m)
- KMP总时间复杂度O(m+n)
- 此next构造仍有缺陷，如aaaab的next序列为0，1，2，3，4。但实际0，0，0，0，4更优
- 这是因为a匹配失败后，向前依然跳到了一个相同的a，那必然失败
```c++
//next优化，若第i项与第next[i]项相同，则可以将next[i]调整为next[next[i]];
void get_next_pro(SString T,int next[])
{
	int i=1,j=0;
	next[0]=-1;
	while(i<T.length)222
	
	{
		if(j==-1||T[i]==T[j])
		{
			i++;
			j++;
			next[i] = (T[i]==T[j]) ? next[j] : j;
		}else{
			j=next[j];
		}
	}
}
```

## 链式存储
- 顺序串的插入和删除不方便，采用单链表可简化这个问题
- 链式串存储中，一个节点可以存储多个字符。由于串长不一定是节点大小整数倍，因此尾节点空置的字符位置可以用不在字符集中的元素（如#）占位。这样定义的串存储结构称为块链结构
```c++
#define CHUNKSIZE 80
struct chunk{    //块结构
	char ch[CHUNKSIZE];
	chunk* next;
};

struct string{
	chunk *head,*tail;
	int length;
};
```
- 串的链式存储占用空间大且操作不如顺序灵活，但在链接、插入等操作上有优势