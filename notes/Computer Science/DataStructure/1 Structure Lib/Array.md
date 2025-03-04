# ADT
---
- 数组是由类型相同的数据元素构成的有序集合，是线性表的推广
- 特点：结构中的元素本身可以是具有某种结构的数据，但属于同一类型
- 二维数组可以看作数据元素是线性表的线性表
- 数组一旦被定义，其维数和维界不再改变


# 顺序存储
---
- 由于数组可能是多维的，而存储单元是一维的，因此语言会约定采取行主序还是列主序
- C、cpp、java、python都采取行主序
- 若有二维数组$A_{m*n}$则其元素地址由$LOC(x,y)=L(0,0)+(n*x+y)*sizeof(DataType)$确定
- 其中$LOC(0,0)\triangleq Base$为基地址

# 特殊矩阵的压缩存储
---
1. 对称矩阵
	- 对于n阶对称矩阵（下标从1开始），可以为每一组对称元分配一个空间
	- 假设以一维数组$sa\left[ \frac{n(n+1)}{2} \right]$存储，则$sa[k]$与$a_{ij}$有对应关系
$$
k=\begin{equation}
\left\{
\begin{aligned}
\frac{i(i-1)}{2}+j-1 \quad i\geq j\\
\frac{j(j-1)}{2}+i-1 \quad i<j
\end{aligned}
\right.
\end{equation}
$$
2. 三角矩阵
3. 对角矩阵