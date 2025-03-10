

# 定义
- 广义表是线性表的推广，也称列表，广泛应用于LISP语言
- 广义表一般记作 $LS=(a_{1},a_{2},\dots a_{n})$
- 其中LS是广义表的名称，n是其长度
- 在线性表中，$a_{i}$只限于单个元素，而广义表中$a_{i}$可以是单个元素也可以是另一个广义表
- 分别称为广义表LS的**原子**和**子表**
- Eg：
	- A=( )      *A是一个空表
	- B=(e)     *B只有一个原子e，长度为1*
	- C=(a,(b,c,d))      *C的长度为2，两个元素分别为原子a，和子表(b,c,d)*
	- E=(a,E)    *这是一个递归的无限表，相当于$E=(a,(a,(a,\dots)))$*


# 操作
---
- 取表头：第一个元素
- 取表尾：除表头外，其余元素构成的表

# 存储结构
---
## 链式存储结构
### 头尾链表
- 一个表可分解为表头和表尾
- 构造两种节点
	- 表节点：标志域、表头指针域、表尾指针域
	- 原子节点：标志域、值域
```c++
typedef enum{ATOM , LIST} ElemTag;    //标志(枚举)：0代表原子，1代表子表
typedef struct GNode{
	ElemTag tag;                       //标志域
	union{                             //共用体
	AtomType atom;                     //原子值域
	struct{struct GNode *hp,*tp;}ptr;  //指针域：包括头表指针和尾表指针
	}
	GNode *next;
}*GList；
```

### 扩展线性链表
