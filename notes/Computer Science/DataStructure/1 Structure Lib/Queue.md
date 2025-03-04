




# 队列
---
- 队列是一种先入先出的有序队列
- 队头front
- 队尾rear

## 操作
- 创建
- 入队
- 出队
- 判满
- 判空
- 查找
- 删除


## 顺序存储
- 利用数组实现队列
```
#define MaxSize <最大元素个数>
struct SeqenceQueque{
	ElemType *base;     //基地址
	int front;          //队列头
	int rear;           //队列尾
};
```


### 循环队列
- 在入队出队时，可能形成队尾到底无法再插入，但队首前有空余的情形，即**假溢出** 
- ![[队列]]
- 此时我们可以将队列成环，将下标模Size
- 但此时front == rear无法判断队列的空满，有两种解决办法
	- 空置一个位置，即n元数组仅用n-1个位置。此时front == rear为空，front == rear -1 (mod Size) 为满
	- 增加额外标记来判断空满，例如增加flag在入队时设置为1，在出队时设置位0，则front == rear 在flag=1时是满，在flag=0时是空


## 链队
- 链表头尾都可以做插入操作，但只有链表头适合做删除
- 链表头做队首，链表尾做队尾
```c++
//节点
struct Node{
	ElemType Data;
	Node* next;
};
//链队
struct LLQueue{
	Node* rear;    //队尾
	Node* front;   //队首
};
```