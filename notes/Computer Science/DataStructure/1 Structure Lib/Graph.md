


# 定义
---
- 图G由点集V(vertex)和边集E(edge、arch)组成
	- V是顶点的有穷非空集合
	- E是V中顶点对的集合
	- E可以为空集，此时G只有顶点无边


# 术语
---
- 有向图：若E中的顶点对有序，则G是有向图
- 无向图：若E中的顶点对无序，则G是无向图
- 同构：若两个图的顶点一一对应，且边对应相等，则称同构
- 子图：若对于图$G=(V,E),G'=(V',E')$有$V'\subseteq V$且$E'\subseteq E$则称G‘是G的子图
	- 支撑子图：$V=V'$
- 相邻、邻接**adjacent**：若两个顶点间有边，则称它们相邻
- 关联：若顶点v是边e的一个端点，则称v和e关联
- 环：若顶点v有从自身到自身的边，则称此边为环
- 平行边：若连结两个顶点的不止一条边，则称这些边为平行边
- 简单图：若一个图没有环和平行边，则称简单图
- 完全图：任意两个顶点相邻
- 阶：顶点个数
- 度：与一个顶点关联的边数
	- 入度：有向图中，指向此点的边数
	- 出度：有向图中，指离此点的边数
- 正则图：每个顶点度数一致
- 链、通道**channel**：相邻顶点的序列
	- 闭链、闭通道：首尾相连
- 迹**trace**：无重复边的链
	- 闭迹：首尾相连
- 路**path**：无重复顶点的迹
- 圈、回路：首尾相连的路
- k部图：若图G的顶点可以划分为k个集合，且一个集合内相邻顶点，则称G是k部图
- 偶图：2部图
- 连通：若两个顶点间存在路，则称它们连通
- 连通图：任意两个顶点间存在路
- 连通分量：图的极大连通子图
- 强连通：在有向图中，若两个顶点间存在双向路，则称它们强连通
- 强连通图
- 强连通分量
- 生成树：连通图的极小连通子图，含有图的全部顶点，但无圈。任意添加一条边都会使之有圈
- 有向树：有一个顶点入度为0，其余顶点入度为1
- 生成森林：若干棵有向树，含有图的全部顶点


 - 稀疏图：边很少   $e<n\log_{2}n$
 - 稠密图：边很多

# 操作
---
- 构造
- 析构
- 查找
- 查询
- 赋值
- 返回邻接
- 删除
- 插入
- DFS
- BFS


# 存储结构
---
## 边列表
- 用数组来储存所有顶点
- 用数组来储存所有边

## 邻接矩阵
- 假设有n个顶点
- 构造$n\times n$矩阵，其中$a_{ij}$若编号为i和j的顶点间有边则填入1，否则填0
	- 若边有权，可以相邻填权，不相邻填$\infty$
- 优点：
	- 判断是否相邻
	- 计算度
- 缺点：
	- 插入、删除顶点 O(n)
	- 统计边的数目 $O(n^{2})$
	- 占用空间大，尤其是稀疏图


## 邻接表
- 为图的每个结点建立单链表，记录其相邻顶点
- 表头结点表：以结构数组储存所有结点，分为数据域、链域，其中链域指向结点的邻接链表
- 边表：由n个边链表组成。分为邻接点域、数据域、链域
- 若为有向图，则分为邻接表和逆邻接表
```c++
struct ArcNode{        //边结点
	int adjV;
	ArcNode* next=NULL;
	InfoType info;
};
struct VNode{         //顶点结点
	DataType data;
	ArcNode* adj=NULL;   //链表头
}；
struct Graph{
	VNode* V;
	int Vnum,Enum;
}
```

- 优点：
	- 便与插入和删除结点
	- 统计边数O(v+e)
	- 空间效率高
- 缺点：
	- 判断邻接 O(v)
	- 计算度 

## 十字链表
- 有向图的链式存储结构，可以看成邻接链表+逆邻接链表
```c++
struct ArcNode{
	int headvex     //边头
	int tailvex     //边尾
	ArcNode* hlink  //边头相同的链中下一个结点
	ArcNode* tlink  //边尾相同的链中下一个结点
};
struct VexNode{
	DataType data;
	ArcNode* hfirst;  //入链的头结点
	ArcNode* tfirst;  //出链的头结点
};
```

## 邻接多重表
- 无向图的链式存储结构，与十字链表类似
- 在邻接表基础上优化对边的操作
```c++
struct ArcNode{
	int iVex,jVex;       //边连结的两个顶点
	ArcNode *ilink,*jlink    //相邻i的链，相邻j的链
	InfoType info;       //储存信息
	int mark;            //标志域：是否已被搜索
};
struct VexNode{
	DataType data;
	ArcNode* first;
}
```



# 遍历
---
## DFS
- 利用递归
- 从某个顶点出发
	- 若有邻接点，访问第一个邻接点，重复直到没有邻接点或邻接点都被访问
	- 返回，重复直到有未被访问的邻接点
	- 访问下一个邻接点
- 根据访问顺序产生的序列称为深度优先生成序列
- 以第一个访问的顶点为根，按访问顺序构造孩子，可以得到深度优先生成树
```c++
//DFS遍历连通图   伪码
bool visited[Vnum]={0};        //标记顶点是否被访问呢
void DFS(Graph G,Vertex v)
{
	access(v);     //访问此顶点
	visited[v.index]=1;   //标记此顶点
	for(Vertex w=FirstAdjVex(G,v);w!=NULL;w=NextAdjvex(G,v,w)){ //循环遍历V的邻接
		if(!visited[w.item]) DFS(G,w);
	}
	return ;
}

//邻接矩阵实现
void DFS_AM(Graph G,int v)
{
	cout<<v;                //访问
	visited[v]=1;	        //标记
	for(w=0;w<Vnum;w++){
		if(a[w][v]&&visited[w])    //若w与v相邻且w未被访问，进入w
			DFS_AM(G,w);
	}
	return ;
}

//邻接表实现
void DFS_AL(Graph G,int v)
{
	cout<<v;
	visited[v]=1;
	ArcNode* p=G.V[v].adj;
	while(p){
		if(!visited[p->adjV]) DFS_AL(G,p->adjV); //若邻接未被访问，则进入
		p=p->next;
	}
	return ;
}
```
- 邻接矩阵：$O(n^{2})$
- 邻接表：$O(n+e)$


## BFS
- 利用队列
- 从某个顶点出发
	- 依次访问所有邻接顶点
	- 依次访问所有邻接顶点的邻接
```c++
//BFS遍历连通图  伪码
void BFS(Graph G,vertex v)
{
	access(v);
	visited[v.item]=1;
	Queue<vertex> Q;
	Q.enqueue(v);
	vertex tem;
	while(!Q.empty()){
		tem=Q.dequeue();
		for(vertex u=FirstAdjVex(G,v);u!=NULL;u=NextAdjVex(G,v,u)){
			if(!visted[u.index]){
				access(u);
				visted[u.index]=1;
				Q.enqueue(u);
			}
		}
	}
}
```


# 最小生成树
---
## 概念
- 对于n个顶点的网，有多种生成树，其中代价最小的称为最小代价生成树(Minimum Cost Spanning Tree, MCST)，简称最小生成树(Minimum Spanning Tree, MST)
- MST性质：设$N=(V,E)$是一张网，$(u,v)$是其中代价最小的一条边，则最小生成树中一定含有这条边
	- 反证法

## Prim算法
- 设$N=(V,E)$是网，构造一个集合U，任取一点放入U中，记最小生成树边集为TE
	- 寻找U与V-U间权值最小的边，将其加入TE中，将相邻点假如U中
	- 重复直到U=V，共n-1次，因为有n-1条边
- **加点法**
```c++
//邻接矩阵储存图
//结构数组储存U-V集合中每个顶点到U中的最小代价
struct{
	Vertex adjvex;
	CostType cost;
}closedge[G.Vnum];   //长度为顶点个数的数组
void MiniSpanTree_Prim(Graph G,Vertex u){
	//从u出发
	int k=u.index;                //k是u的编号
	//初始化closedge为u的邻接
	for(int i=0;i<G.Vnum;i++)
		if(i!=k) closedge[i]={u,G.arcs[i][k]};
	//循环n-1次载入n-1条边
	for(int i=1;i<G.Vnum;i++)
	{
		k=MinCost(closedge);   //closedge[k]的cost是最小的
		TE.append((G.V[k],closedge[k].adjvex));   //在TE中加入这条边
		//更新closedge
		//删除closedge[k]
		//对于closedge中的每个结点，新增了与k邻接的可能，比较cost，选择更小者
		closedge[k].cost=0;
		for(int j=0;i<G.Vnum;i++)
		{
			if(G.arcs[j][k]<closedge[j].cost) closedge[j]={k,G.arcs[j][k]};
		}
	}
}
```
- $O(n^{2})$ 


## Kruskal算法
- 将边按权值大小从小到大排序
- 初始T中只有网N的全部顶点，没有边，所有顶点互不连通
	- 在边中选择端点不连通的，权值最小的加入T中
	- 重复n-1次
- **加边法**
```c++
//边列表储存
//结构体数组edges储存边
struct{
	Vertex head;
	Vertex tail;
	ArcType cost;
}Edge[Enum];
//数组Vexset表示各个顶点所属的连通分量，初始时每个顶点自成连通分量
int Vexset[Vnum];

void MiniSpanTree_Kruksal(Graph G)
{
	Sort(Edge);
	for(int i=0;i<G.Vnum;i++) 
		Vexset[i]=i;      //初始化Vexset
	for(int i=0;i<G.Vnum;i++)
	{
		Vertex v1=Edge[i].head;
		Vertex v2=Edge[i].tail;
		int vs1=Vexset[v1.index];    //v1的连通分量
		int vs2=Vexset[v2.index];    //v2的连通分量
		if(vs1!=vs2)                 //v1、v2不属于同一连通分量
		{
			T.append(Edge[i]);       //加入边
			for(int j=0;j<G.Vnum;j++)  
				if(Vexset[j]==vs2) Vexset[j]=vs1; //将v2的连通分量合并入v1的连通分量
		}
	}
}
```
-  $O(e\log_{2}e)$ 
-  适用于稀疏图


# 最短路径
---
- 源点：路上第一个顶点
- 终点：路上最后一个顶点


## Dijkstra算法（迪杰斯特拉）
- 求从某顶点开始到其余顶点的最短路径
- 动态规划、记忆化搜索
- 带权有向图$G=(V,E)$，源点$v_{0}$
- 集合S标记结点是否被计算出最短路径长度
- 集合Dist记录已求出的路径长度，不一定是最短的
- 集合Prior记录结点在路径中的前驱
	- 关注最新标记结点的邻接，若新的路径长度小于已有路径长度，则更新它们的Dist和前驱
	- 在Dist中取未标记最小，设置为已标记，载入前驱
```c++
//用带权邻接矩阵储存图G
struct{
	int Vnum;
	int Enum;
	int V[Vnum];
	int E[Vnum][Vnum];
}G;
//宏INFTY表示无穷大，即两个顶点不相邻
//一维数组S[]记录顶点是否被计算，1表示被计算，0表示否
//一维数组Prior[]记录被计算顶点的前驱，以-1表示未被计算
//一维数组Dist[]记录路径长度，以INFITY表示未被计算

void Dij(Graph G,int v0){
	int n=G.Vnum;
	bool S[n]={0};
	int Prior[n];
	int Dist[n];
	//初始化
	S[v0]=1;
	for(int i=0;i<n;i++)
	{
		if(G.E[v0][i]<INFITY){
			Prior[i]=v0;
			Dist[i]=G.E[v0][i];
		}else{
			Prior[i]=-1;
			Dist[i]=INFTY;
		}
	}
	//dp
	for(int i=1;i<n;i++)   //n-1次循环，标记n-1个点
	{
		int v,min=INFTY;
		for(int j=0;j<n;j++){   //查找未标记最短路径
			if(S[j]==0&&min>Dist[j]){
				v=j;
				min=Dist[j];
			}
		}
		S[v]=1;   //标记为已确认最短路径
		for(int j=0;j<n;j++)   //更新Dist和Prior
		{
			//若结点未被标记，与v邻接，新dist小于原dist，则更新
			if(S[j]==0&&
			   Dist[v]+G.E[v][j]<Dist[j]){
				Dist[j]=Dist[v]+G.E[v][j];
				Prior[j]=v;   
			}
		}
	}
}
```


## Floyd算法（弗洛伊德）
- 求每一对顶点间的最短路径
- 可以调用n次Dij也可以用Floyd，二者时间都是$O(n^{3})$，但Floyd形式更简单
- 二维数组Path记录$v_{i}\to v_{j}$路径上$v_{j}$的前驱
- 二维数组D记录最短路径长
- 初始化D为边（不邻接则INFITY），Path为相邻点或-1
	- 在$v_{i}$和$v_{j}$间插入$v_{0}$，即$v_{i}\to v_{0}\to v_{j}$比较D是否变小，选择更小者，依次更新全部$n^2$条路
	- 插入$v_{1}$，即$v_{i}\to\dots\to v_{1}\to\dots\to v_{j}$，其中$v_{i}\to\dots\to v_{1}$和$v_{1}\to \dots \to v_{j}$是上一步求出的最短路径。比较D是否变小，选择更小者，依次更新
	- 重复更新n-1次
```c++
void Floyd(Graph G)
{
	//初始化
	for(int i=0;i<G.Vnum;i++)
	{
		for(int j=0;j<G.Vnum;j++)
		{
			D[i][j]=G.E[i][j];
			if(D[i][j]<INFTY) Path[i][j]=i;
			else Path[i][j]=-1;
		}
	}
	for(int k=0;k<G.Vnum;k++)
	{
		for(int i=0;i<G.Vnum;i++){
			for(int j=0;j<G.Vnum;j++){
				if(D[i][k]+D[k][j]<D[i][j])
				{
					D[i][j]=D[i][k]+D[k][j];
					Path[i][j]=Path[k][j];
				}
			}
		}
	}
}
```

# 拓扑排序


# 关键路径