
# 1 Preface
---
- Course Outline
	- Relational Databases
	- Storage
	- Query Execution
	- Concurrency Control
	- Database Recovery
	- Distributed Databases
	- Potpourri

- Layers of DBMS
	- Query Planning：服务
	- Operator Execution：符号
	- Access Methods：访问
	- Buffer Pool Management：内存
	- Disk Manager：硬盘


# 2 Lecture 1: Relational Model
----
## 2.1 Background
- 普通文件系统的不足
	- Data Integrity
		- 无法检测非法信息
		- 修改不方便
	- Implementation
		- 并发修改
		- 异架构迁移
	- Durability
		- 可用性
- 什么是Database：
	- Organized collection of inter-related data that models some aspect of real life
- 什么是Database Manage System, DBMS：APP与数据库之间的中间层
	- A database management system (DBMS) is the software that allows applications to store and analyze information in a database.
- Data Model：a collection of concepts for describing the data in a database
	- 数据模型，概述数据的组织方式，例如Relational Data Model
- Schema：defines the structure of data for a data model.
	- Model的具体实现，表的实际构造

## 2.2 Relational Model
### 2.2.1 Relation
- Relation
	- an unordered set that contain the relationship of attributes that represent entities
	- Relation与Table等价，代表一个表，表里有一个个实例（entities）
- Tuple：A tuple is a set of attribute values (aka its domain) in the relation.
	- 元组是组成表的单位
- Primary Key：主键
- Foreign Key：外键
- Constraints：对于属性的约束
### 2.2.2 Independence
- 物理存储方式由DBMS实现，用户无需关心
- DBMS会根据OS等找到最佳存储策略
- 通过高级语言访问，DBMS进而找到最佳策略

## 2.3 Relational Algebra
- 关系运算 Operations
	- 选择 Select
	- 映射 Project
	- 差集 Difference
		- 要求属性名相同
	- 交集 Intersect
		- 要求属性名相同
	- 并集 Union
		- 要求属性名相同
	- 笛卡尔积 Product
	- 连接 Join
- 关系代数是过程式描述
- 关系代数是查询的基础
- SQL是声明式语言

## 2.4 Alternative Data Models
- Document Data Model：Mongo DB
- Vector Data Model


# 3 Lecture 2 Modern SQL
---
## 3.1 Relational Language
- 大致可以分为三类
	- Data Manipulation Language, DML
	- Data Definition Language, DDL
	- Data Control Language, DCL
- others
	- View defination
	- Integrity & Referential Constraints
	- Transactions
- SQL is based on **bags** instead of sets, allowing duplicates

## 3.2 Aggregation
- 什么是聚合：
	- 从已有数据获取更高层次信息
- Aggregation Function
	- Function that return a single value from a bag of tuples
- Ex:  AVG, MIN, MAX, SUM, COUNT
- SQL聚合
	- GROUP BY:  Any non-aggregate value must appear in GROUP BY clause
	- HAVING：对聚合后的表筛选
	- 几种Join
		- LEFT JOIN：返回左表的所有记录和符合条件的右表记录
		- RIGHT JOIN：同理
		- INNER JOIN：只返回符合条件的两表记录
## 3.3 String
- 不同系统对于单双引号和大小写要求不同
- 百分号`%`用来通配字符串，下划线`_`用来通配字符

## 3.4 Date/Time
```sql
SELECT NOW()

SELECT DATE('2025-10-1')

SELECT EXTRACT(day from DATE('2025-10-1'))
```
- 日期算法在不同数据库中实现不同，这里不展开

## 3.5 Output control
- 输出流控制
```sql
order by
limit
fetch 
offset
```

## 3.6 Redirection
- 输出可以重定向到表中
```sql
create newTable as
 select ...
 
 select ... into ...
```


## 3.7 Window functions
- 以滚动的窗口进行聚合计算
```sql
SELECT FUNC-NAME() OVER ...
```
- OVER: how to slice data 
- 一些窗口函数
	- ROW_NUMBER() 
	- RANK():
```sql
select *, RANK() OVER (PARTITION BY cid ORDER BY grade ASC) AS rank FROM enrolled
```
- 这里使用`PARTITION BY`而非`GROUP BY`，前者用于窗口，后者用于聚合

## 3.8 Nested Queries
```sql
SELECT sdi, 
	(SELECT name FROM student as s WHERE s.sid = e.sid) as name
	FROM enrolled as e
	
	
SELECT name FROM student as s
	WHERE sid IN (
		SELECT sid FROM enrolled WHERE cid = '15-445')

SELECT name FROM student as s
	WHERE sid = ANY (
		SELECT sid FROM enrolled WHERE cid = '15-445')
```
- nested query可以用join优化 query optimization
- ALL：子查询的所有行都要满足
- ANY、IN：子查询其中至少一行满足
- EXIST：子查询至少一行满足，且不比较？
- 现在我们想查询至少加入一门课的学生的最高的id
```sql
SELECT MAX(e.sid), s.name
	FROM enrolled as e, student as s
WHERE e.sid = s.sid
```
- 上述聚合是错误的，因为s.name不是组别，且`group by s.name`毫无意义
- 修改为嵌套查询
```sql
SELECT sid, name FROM student
WHERE sid = ANY(SELECT MAX(sid) FROM enrolled)
```
- 从外向内

## 3.9 Lateral Joins
- 横向连接
- 内部查询可以引用外部查询的元组，但外部查询不能引用内部查询的元组
- 横向连接允许一个查询访问其之前的嵌套查询的元组
```sql
SELECT * FROM
  (SELECT 1 AS x) AS t1,
  LATERAL (SELECT t1.x+1 AS y) AS t2
```

## 3.10 Common Table Expression, CTE
- 创建临时表
```sql
WITH cteName (col1, col2) as (
	SELECT 1,2
)
```


# 4 Lecture 3&4&5：Database Storage -- Disk Manager
---
## 4.1 Storage Hierarchy
- 从上到下
	- CPU Register
	- CPU Cache：高速缓存
	- DRAM：内存
	- SSD：固态硬盘
	- HDD：Hard Disk Drive   通常指磁盘
	- Network Storage
- 前三者
	- Volatile
	- Random Access
	- Byte-Addressable
- 后三者
	- Non-Volatile
	- Sequential Access
	- Block-Addressable
- 我们用**memory**指代内存、缓存等空间，**disk**指代硬盘、磁盘等空间
## 4.2 System Design Goals
- 总是假设数据在硬盘上而非内存中
- 小心进行数据处理，避免大规模停滞和性能下降
- 尽可能保证顺序读写


## 4.3 Disk-Oriented DBMS Hierarchy
- Disk：硬盘上存储了数据库文件（Database File），每个文件有一个目录（Directory）和多个页（Pages）
- Memory：内存上有缓冲池（Buffer Pool）
- Execution Engine：执行引擎调度内存和硬盘
![[Pasted image 20251229163502.png]]

## 4.4 Disk Storage
### 4.4.1 Page
- 在硬盘上，数据以**页, Page**的形式组织
- different notions of **page**
	- Hardware Page (4KB usually)：硬盘原子IO单位，要么都写要么都不写
	- OS Page 
	- Database Page
- A **DataBase Page** is a fixed-size block of data on disk
	- 通常一个page内的内容是一类的
	- 有些系统要求page是自包含的（self-contained）
		- 容灾
	- 每个page有唯一标识符（page ID）
- Database Page如何选择大小
	- 数据特性
	- 工作负载
		- 页过大，修改时读入大量无用数据
		- 页过小，无法存放数据

### 4.4.2 How To Organize Pages On Disk
- Page Storage Architecture
	- Heap File Organization
	- Tree File Organization
	- Sequential/Sorted File Organization, ISAM
	- Hashing File Organization
- Heap File是存储了pages的无序集合
	- 有CRUD接口
	- 有迭代器
	- 需要额外储存元数据来追踪文件地址和计算空余空间
- Page Directory维护Pages的索引
	- Page Directory本身也是一个Page
	- 写Page时同步更新Page Directory
- Meta-Data
	- 剩余空间
	- 空页地址 Free Space Map， FSM
	- 页类型

### 4.4.3 Inside a Page
- Header：存储内容相关的meta-data
	- page size
	- checksum
	- DBMS version
	- Transaction Visibility
	- Compression Encoding Meta-data
	- Schema Information
	- Data Summary/Sketches
- Data
### 4.4.4 Layout
- Page Layout
	- Tuple-oriented Storage：元组
	- Log-structured Storage
	- Index-organized Storage：树形

## 4.5 Inside Page
### 4.5.1 Tuple-Oriented Storage
- 想法：header存储tuple数量，tuple依次排列
	- 如何删除？
	- tuple长度不一致？
- The most common layout cheme is **slotted pages**
- Slotted Pages：槽页   
	- slot array：记录每个槽的大小
	- header：槽的个数、最后一个槽的位置
	- slot：槽从page的尾部开始向前排列，每个槽存储一个tuple
- 删除时，只需要将slot array中对应的槽标记为空即可
	- 有的系统会在删除后进行压缩保证槽是连续的
- Record ID：
	- DBMS为每个tuple分配record ID来标识它实际的物理位置
	- 这个ID不应当在应用层被使用
- 一些问题
	- Fragmentation: Slot Array从前向后扩展，Slot从后向前扩展，中间往往不能对齐而留下空隙
	- Useless Disk IO: DBMS必须读取整个Page来更新一个Tuple
	- Random Disk IO：修改10个不同页面的Tuple需要读取10个不同Page

### 4.5.2 Log-Structured Storage
- 以日志形式存储数据
- Log中的每条都是一次PUT或DELETE操作，形如`PUT< key101, data >`
- MemTable
	- 内存中的树形结构
	- 发生修改时，记录先被存入MemTable的叶子
	- 当内存存不下时，将MemTable中的记录按键序形成顺序表，形成SSTable存入硬盘
	- MemTable以树的形式组织是为方便查找
- SSTable
	- 硬盘中的顺序表
	- 通常形成层级结构，第一层满时，将第一层的所有SSTable合并（剔除冗余记录）为更大的SSTable存于第二层。以此类推形成多层。每层从新到旧。
- SSTables形成LSM Tree，log-structured-Merge-Tree
- 读操作
	- 首先在MemTable中找Key
	- 没找到再去SSTable中按层级从上到下二分查找
- 为了优化读操作，我们在内存中维护Summary Table， 记录SSTable的信息
	- Min/Max Key per SSTable
	- Key Filter per Level
- compaction：对SSTable压缩
	- 归并排序，总是选择更新的记录
- 优缺点
	- pros
		- 硬盘上没有Random Access，都是Sequential
		- 写操作很快
	- cons
		- Write-Amplification：一条记录在生命周期中会经历多次移动
		- Compaction is Expensive：硬盘上的归并很昂贵
		- Slow Read

### 4.5.3 Index-Organized Storage
- 在前面两种结构中，我们都依赖索引表查找Record ID
- 如果直接让索引指向数据呢
- 容易读，不容易写，与log-structure相反
- 仍然使用slot-page，但tuples和slot-array按键排序了
- Read
	- 在B+Tree中找到叶子，索引到Page
	- 在Page的slot-array中二分查找


## 4.6 Inside Tuple
- Tuple 是一串字节序列
	- Header
	- Content
- DBMS interpret the bytes into attribute types and values.
- Word-Alignment：元组需要字对齐
	- padding *多数采用这个*
	- reordering
- 浮点舍入误差是不能接受的
	- 不采用IEEE标准，而是fix-point numeric
- Null Data Type的三种方式
	- Bitmap Header：用header记录此处是否为空，这是行存储最常用的
	- Special Value：用特殊值，例如INT32_MIN，这是列存储最常用的
	- Per Attribute Null Flag：为每个属性都设置null flag，这会破坏对齐


## 4.7 Column Storage
- 列存储是Tuple-Oriented Storage的一种
### 4.7.1 Database Workloads
- On-Line Transaction Processing, OLTP
	- Fast operation that only read/update a small amount of data each time
- On-Line Analytical Processing, OLAP
	- Complex queries that read alot of data to compute aggregates
- Hybrid Transaction + Analytical Processing
	- OLTP + OLAP
- 考虑两个维度
	- Workload Focus: Wirte-Heavy/Read-Heavy
	- Operation Complexity

### 4.7.2 Storage Models
- Storage Model阐述了一个DBMS是如何实现元组存储和内存管理的
	- N-ary Storage Model, NSM：i.e. Row Storage
	- Decomposition Storage Model, DSM: i.e. Column Storage
	- Hybid Storage Model, PAX
#### 4.7.2.1 NSM
- Store all attributes for a single tuple contiguously in a single page.
- 适合OLTP
	- 查询总是针对少量元组
	- write-heavy，插入快
- 不适合OATP
	- 针对大量元组的特殊属性的查询会读取很多无用数据
	- 难以压缩
#### 4.7.2.2 DSM
- Store a single attribute for all tuples contiguously in a block of data.
- Seperate page per attribute with a dedicated header for meta-data about entire column
- Tuple identification: 如何匹配属性和元组
	- Fixed-length offsets：每列都一样长
	- Embedded Tuple IDs：*Don't do this*
- Variable-length Data?
	- Padding to fixed-length is wasting
	- Dictionary compression
- Pros
	- 减少无效数据IO
	- 相同的列长度能够加快顺序遍历，高效查询
	- 值域相同，更好压缩
- Cons
	- slow for point queries, inserts, updates and deletes
- 行存储预写，列存储压缩

#### 4.7.2.3 Pax
- Partition Attrbutes Across, PAX
- A hybrid storage model that vertically partitions attributes within a database page
- layout
	- 先按tuple分组形成多个row groups
	- 每个row groups按列切块形成column chunks
	- 每个row groups内部按列存储
	- 所有row groups顺序存储
	- 行组比page大的多
- 通常说的列存储都会采用PAX
- PAX在列存储的基础上，让行相近的元组在物理上更相近


#### 4.7.2.4 Compression
- 利用压缩提高一次读取的数据量
- trade-off between compression versus computation
	- 压缩会提高计算量解码
- Goals
	- 结果要是定长的
		- Must produce fixed-length values
	- 在查询过程中，解压缩尽可能滞后
		- late materialization
	- 数据不丢失
		- lossless scheme
- Granularity
	- Block-level
	- Tuple-level
	- Attribute-level
	- Column-level
- Naive MYSQL Block-level Compression
	- 记录mod log修改日志，压缩page
	- 修改直接加入mod log，读取先看mod log
	- 解压缩时清空mod log完成修改
- 压缩后的数据没有含义，无法实现late materialization
- 数据值域不同，压缩效果差
- 我们希望压缩后的数据也能直接操作
- Columnar Compression
	- Run-length Encoding：游程编码
		- 压缩连续相同值
		- 为了取得最佳压缩效果，需要排序
	- Bit-Packing Encoding
		- 若值有上界，则可以删去无效位，通常用于unsigned int
		- 通过左移可以轻松实现
		- 可能出现个别值，特殊处理：pathcing/mostly encoding
	- Bitmap Encoding
		- 将每个值作为bitmap的一位
		- 基数很大时该方法失效
		- 适用于稀疏
	- Delta/Frame-of-Reference Encoding
		- 差分
		- 可以用RLE进一步压缩
	- Incremental Encoding
	- Dictionay Encoding
		- 最好先排序，使键和值偏序相同，支持范围查找


# 5 Lecture 6：Database Storage --- Buffer Pool Manager
---
- Spatial Control
	- 存的连续
- Temporal Control
	- 读取到内存的数据用的尽量久
	- 尽量少读取
- Other Memory Pools
	- Sorting + Join Buffers
	- Query Caches
	- Maintenance Buffers
	- Log Buffers
	- Dictionary Caches

## 5.1 Buffer Pool Manager
- Frame：缓冲池中的帧，对标Disk上的Page
- Dirty Page：脏页，被修改的页不会立即写入硬盘，而是buffered
	- write-back cache
	- write-through cache
- Page Table：从PageID到FrameID的映射
	- 接受一个PageID，缓冲池索引对应的FrameID，没找到则从硬盘上读取
- Additional meta-data per Page
	- Dirty Flag
	- Pin/Reference Counter
	- Access Tracking Information

### 5.1.1 锁 Locks vs. Latches
- 在数据库中，lock的含义与进程中的lock不同，通常表示更高层次的锁
- Locks：应对外部事务请求
	- Protects the database's logical contents from other transactions
	- Held for transaction duration
	- Need to be able to rollback changes
- Latches: 闩锁  Mutex 互斥锁  DBMS内进程管理
	- Protects the critical sections of the DBMS's internal data structure from other threads
	- Held for operation duration
	- Do not need to be able to rollback changes

### 5.1.2 Page Table VS. Page Directory
- Page Directory
	- on disk
	- mapping from pageID to page location in db files
- Page Table
	- in memory
	- mapping from pageID to a copy of the page in buffer pool frames


## 5.2 Why not MMAP
- OS mmap
	- 将硬盘上的文件映射位虚拟内存上 
- Problems
	- TX safety：OS会自动清理脏页，但DBMS需要保证事务完成后脏页才被写入，否则无法知道哪一部分被写入了
	- IO stalls：DBMS不知道内存中有哪些Page。OS会在页错误时停止进程
	- Error Handling：Difficult to validate pages. Any access can cause a SIGBUS.
		- MMAP将外存文件映射为虚拟内存，DBMS需要在所有访问虚拟内存的地方做异常处理。反之，只需要在Disk IO的地方异常处理。
	- Performance Issues



## 5.3 Disk IO Scheduling
- OS通过排序和批处理来最大化IO带宽，但OS不知道请求的优先级，OS至多在线程级别设置IO优先级
- DBMS维护自己的硬盘调度器来对硬盘IO排序，通常建议关闭OS IO
- priority factors
	- Sequential VS. Random IO
	- Citical Path Task VS. Background Task
	- Table VS. Index VS. Log VS. Ephemeral Data
	- Transaction Info
	- UserBased SLAs
- IO uring：异步处理
- OS有自己的Page Cach，DBMS通常设置绕过OS Page Cache的Direct IO
- fsync's problem
	- fsync可能将数据写入硬盘缓存后返回成功，此时硬盘停电将导致数据丢失
	- fsync在异常时会清除脏页标记，使得第二次调用fsync没有作用却返回成功

## 5.4 Replacement Policies
- When the DBMS needs to free up a frame to make room for a new page, it must decide which page to evict from the buffer pool.
- Goals
	- Correctness
	- Accuracy
	- Speed
	- Meta-data overhead
### 5.4.1 LRU
### 5.4.2 CLOCK
- LRU的近似
- 不记录时间戳，保留一个bit标识 Reference Bit
	- 将所有page组织为一个圈，clock hand指针指向某一个文件
	- 当page被访问时，RB置1
	- 清理时，若指针指向的page.RB为1，则置0，指针指向下一个
		- 否则清理当前page
- Sequential Flooding：遍历所有页的查询会污染缓冲池
- 在OLAP中，最近访问的却是最佳驱逐选项
- LRU和CLOCK只记录最近访问的，而没记录frequency

### 5.4.3 LRU-K

### 5.4.4 Localization
- 为特定查询维护特定缓冲池，避免缓冲池污染

### 5.4.5 Priority Hints
- Query提示需要访问的Page

### 5.4.6 Dirty Pages
- Fast Path：非脏页，直接驱逐
- Slow Path：脏页，先写回
- 驱逐重要非脏页和写回脏页的平衡
	- 无法解决，只能缓和
- Background Writing：后台写入脏页

## 5.5 Buffer Pool Optimizations
- Multiple Buffer Pools
- Pre-fetching：预取
- scan-sharing：将相关的查询连接
- Buffer Pool Bypass：局部化顺序遍历


# 6 Lecture 7：Access Method
---
## 6.1 Hash Table
### 6.1.1 Bg
- Design Decision
	- Data Organization
	- Concurrency
- 不能只考虑时间复杂度，常数matter
- Hash Table
	- Hash Function
		- How to map a large key space into a smaller domain
		- Trade-off between fast vs. collision rate
	- Hashing Scheme
		- How to handle key collisions
		- Trade-off between allocating a large hash table vs. additional instructions to IO keys
### 6.1.2 Hash Func
- Converts arbitrary byte array into a fixed-length code
	- CRC-64
	- Murmur Hash
	- Google CityHash
	- Facebook XXHash
	- Google FarmHash

### 6.1.3 Static Schemes
- Approaches
	- Linear Probing
		- 线性探测
		- 跟踪load factor，在过载时扩大表至两倍
		- 尽量预估load factor，避免重载
	- Cuckoo Hashing
		- 用多个Hash Function确定位置，找一个空的
		- 没有空的踢一个走
#### 6.1.3.1 Linear Probing
- Key/Value Entries
	- Fixed-length K/V
		- 直接存储Key-Value
		- 可以存储该位置的Hash值来加速查询
	- Variable-length K/V
		- Hash表只Hash值和RecordID
		- ‘Key-Value存储在另外的临时表中，由RecordID索引
		- 临时表是查询的私有表，无需恢复机制 
- Delete
	- Tombstone：懒删除
- Non-Unique Keys
	- Separate Linked List
	- Redundant Keys：允许重复键存储
- Optimization
	- 根据数据选择Hash表
	- 维护元数据加速查询
	- 用版本号表示Hash表被删除


### 6.1.4 Dynamic Schemes
- 需要时自动扩展大小
	- Chained Hashing
	- Extendible Hashing
	- Linear Hashing
#### 6.1.4.1 Chained Hashing
- 维护目录 Directory
	- Hash Function先映射到目录
	- 目录存储指向桶链的指针
- 维护桶链 Chain of Buckets
	- 桶链存储Key-Value
- 优化
	- 在链表前维护filter，指示某个键是否存在于链表中

#### 6.1.4.2 Extendible Hashing
- Chained Hash中如果所有键被hash到同一链表上，每次查询都会遍历所有元素
- Extendible Hashing允许桶的拆分，同时避免表的重载
- 允许多个条目指向一个桶
- 目录维护global计数器，指示hash表需要查看hash键的前几位
- 桶维护local计数器，指示桶需要查看hash键的前几位
- 桶在溢出时分裂

#### 6.1.4.3 Linear Hashing
- 维护一个指向桶的指针，该桶会在下次被分裂
	- 任意桶溢出时，分裂指针指向的桶
- 用multiple hashes找到对的桶
- 有不同的溢出标准
- 溢出
	- 申请新的桶
	- 将指针指向的桶的元素rehash
	- hash导致溢出的元素
- 查询
	- 根据指针位置选择hash function


## 6.2 B+Tree
- 自平衡m叉树
	- 查询、顺序访问、插入、删除都是$\mathrm{O(\log _{m}n)}$
	- m称为扇出系数 fan-out
	- 为顺序访问提供强支持
- 结点
	- 元信息
		- 结点类型
		- 空槽数
	- 前指针
	- 后指针
	- key-value
- values
	- recordID：Most Common
	- Tuple Data：Index-Organized Storage
	- Primary Key：二级索引指向主键
- 删除
	- 结点过少，从兄弟借，借不到合并
	- 合并完修改父结点的键
	- 检查父结点缺省
	- 注意无需向上修改所有的键，我们不用保证键等于右子树最小值，只需小于等于即可
- 支持对Composite Index的partial查找
- Duplicate Key
	- Append RecordID
	- Overflow LeafNodes
- Clustered B+Tree
	- page中值的顺序与索引顺序相同，从而顺序遍历叶子时顺序读取page即可
	- 加速顺序访问
	- 这需要reorganize pages
	- 一种替代方式是在顺序遍历索引时先拿到tuple ID而不取tuple，再根据tuple的pageID排序，然后取tuple
- intra-node search
	- linear
		- vector SIMD
	- binary
### 6.2.1 Optimization
- Pointer Swizzling
	- 在结点中存储page指针而非pageID，避免重复buffer pool访问
- Bulk Insert
	- 对一个表键B+树最快的方法是：对键排序，在键链表上建树
- Wirte-Optimized B+T
	- 用日志预写


# 7 Lecture 8：Other Data Structure
---
## 7.1 Blooom Filter
- Probabilistic data structure (bitmap) that answers set membership queries
	- False Negative will never occur
	- False Positive is allowed
- Methods
	- Insert(x)
		- 用若干个hash函数翻转bit
	- Lookup(x)
		- 查看对应bit
- Counting Bloom Filter
	- 用计数代替bit
	- 支持删除
- Cuckoo Filter
	- 类似Cuckoo表
	- 存储指纹而非完整的键
- Succinct Range Filter (SuRF)
	- Immutable
	- 支持范围过滤
	- Tries变体

## 7.2 Skip list
- 类似B+Tree
- 采用倍增思想
- 从链表自底向上建立
	- 底层为原始链表
	- 在上层建立新的链表，起点与原始链表相同，但是每个链接会跨越下层的一个结点，直接链到下下个结点，使该层结点数为下层的一半
- $\mathrm{O(\log n)}$查找、插入、删除
- 多用于in-memory
	- LSM MemTable
- 插入
	- 自底向上，概率选择是否继续向上加结点
	- 建立新的结点
	- 在每层找到合适的位置但不修改指针
	- 建立新结点的层间指针
	- 自底向上建立新结点的后指针
	- 自底向上建立新结点的前指针
	- 该过程无需加锁
- 删除
	- 懒标记
	- 自上向下删除
- Pros and Cons
	- 而更少的内存
	- 插入删除无需rebalance
	- 无法用于硬盘
	- 无法倒序查找

## 7.3 Tries
- Prefix Tree

- Optimization
	- 垂直：单长路径压缩
	- 水平：状态压缩
- 比B+更紧凑


## 7.4 Inverted Index
- Keyword查找：根据内容中的某一部分查找
- Full-text search index：全文搜索系统
- Components
	- Dictionary：Term、Frequency、Pointer to Posting list
	- Posting List
- Xapian：cpp写的库
- Lucene：Java

### 7.4.1 Lucene
- Dictionary
	- Finite State Transducer，FST：有限状态转换机
		- 计算词在字典中的位置
		- 类似于Tries，但边有权重，搜索Key累计的权重就是词在字典中的偏移
- 增量创建字典，后台融合
- 压缩

### 7.4.2 PostgreSql
- Gerneralized inverted index，GIN
- 用B+Tree组织字典，叶子映射到posting list
- PendingList：mod log

### 7.4.3 Vector index
- Embedding
- RAG
- Nearist Neighbor Search
- Inverted File
- Navigable Small Worlds



# 8 Lecture 9: Index Concurrency Control
---
> Concurrency Control Protocol: The correct results for concurrent operation on a shared object

- Correctness
	- Logical Correctness: 操作的语义和结果；线程眼中的世界是什么样的
	- **Physical Correctness: Is the internel representation of the object sound. 物理存储单元如何被安全的访问**
	- 逻辑正确性需要依靠物理正确性实现
- 本节我们关注物理正确性

- 也有一部分数据库认为单线程执行是更好的，例如Redis

## 8.1 Latches Overview
- Locks vs. Latches
	- Locks: Transactions
		- 多个事务并发时，保护数据库的逻辑内容，例如表、元组。保证操作的原子性。
		- 需要支持回滚
	- Latches：Workers
		- 保证数据库内部操作的线程安全
		- 不需要支持回滚
![[Pasted image 20251217190642.png]]

- 锁的模式
	- 读锁、共享锁
	- 写锁、独占锁
- 希望闩锁的实现
	- 占用少量内存
	- 无竞争时能快速获取
	- 去中心化管理
	- 避免使用操作系统原语

### 8.1.1 Implementations
- Test-and-Set Spinlock
	- 定义一个原子变量latch
	- 自旋CAS
	- 这里的TAS和CAS, Compare-and-Swap等价
		- `CAS(&m, a, b)`：检测地址m的值是否为a，若是，则替换为b
	- CAS是硬件层面保证的原语
- Blocking OS Mutex
	- 独占读写
	- ·`std::mutex`：底层是`pthread_mutex_t`，是OS层面的futex，Fast Userspace Mutex
	- 关于futex
		- Futex组成包括内核空间的等待队列和用户空间的原子变量
		- 无竞争下锁的获取不涉及内核，只CAS用户空间的原子变量，性能很高
		- 竞争时陷入内核，会获取系统级别的锁，有系统调用开销，性能低
		- 锁被释放时，系统会唤醒等待的线程
- Reader-Writer Latches
	- 独占写，共享读
	- 需要维护等待队列防止饥饿
		- 若等待队列中有写请求，读请求也会被阻塞
	- `std::shared_mutex`：底层是`pthread_rwlock_t`，再底层是`pthread_mutex_t`和`pthread_cond_t`
- Advanced approaches
	- Adaptive Spinlock
	- Queue-based Spinlock
	- Optimisitic Lock Coupling


## 8.2 Hash Table Latching
- 根据粒度可以分为
	- Block/Page Latch
	- Slot Latch
- 在线性探测时，可以先释放旧锁，再获取新锁，这不会有问题。在B+Tree中不行。
- hash表的resizing一般会用整张表的独占锁


## 8.3 B+Tree Concurrency Control
- B+树中不同线程并发可能导致数据结构出现物理性错误
	- T1请求删除a
	- T1访问A结点，删除了a，此时A低于半满，需要从兄弟B结点拿走b，但此时T1被中断
	- T2请求访问b
	- T2访问B的父节点，判断出b在B结点，在将要访问B结点时T2被中断
	- T1恢复并将b从B结点移动到A结点
	- T2恢复并访问B结点，b不在B结点，出现错误
- 原因分析
	- 每个线程只能看到数据结构的局部内容
- `deschedule`：表示停止调度
### 8.3.1 Latch Coupling
- 闩锁耦合：保障B+树被安全访问和修改的并发协议
	- 获取当前结点的锁
	- 查找下一个访问的子节点
	- 获取子节点的锁
	- 若**安全**，释放当前结点的锁并移动到子节点
- 安全结点：称修改时不会发生分类或合并的结点为安全结点
	- 插入时，非满结点是安全的
	- 删除时，超半满结点是安全的
- 读操作
	- 从根开始向下遍历
		- 获取子节点读锁
		- 移动到子节点
		- 释放父节点读锁
	- 由于读操作不修改结点，因此释放总是安全的
- 写操作
	- 从根开始向下遍历
		- 获取子结点的写锁
		- 移动到子节点
		- 若子节点是安全的，释放持有的所有锁
		- 若子节点不是安全的，继续持有锁
	- 由于写操作在未来可能导致不安全结点分裂或合并，因此需要继续持有经过的所有不安全结点的锁，直到抵达一个安全结点
	- 自顶向下释放锁
		- 从正确性角度讲，自顶向下释放和自底向上释放都是正确的
		- 但顶上的锁包含了更多资源，自顶向下释放能够提高效率
### 8.3.2 Optimistic Latching Coupling
- 闩锁耦合中，写请求总是获取根的写锁，效率降低。而根节点分裂或删除的概率是很低的。
- 乐观锁耦合、Baer-Scholotnick Algorithm
	- 乐观执行，出错回滚
	- 读请求不变
	- 写请求除了叶子外，都获取读锁，若叶子需要分裂或合并，回滚，以悲观方式再运行一遍
- 写操作
	- 从根开始向下遍历
		- 子节点不是叶子
			- 获取子节点读锁
			- 移动到子节点
			- 释放父节点读锁
		- 子节点是叶子
			- 获取叶子写锁
			- 释放父节点读锁
			- 若需要合并或分裂：回滚，以悲观方式运行
- 又称Crabbing Protocol（螃蟹协议）、Latching Crabbing

### 8.3.3 Leaf Link Scans
- 由于B+树的叶子间有链，因此叶子链上的移动会引发竞争
	- T1查询所有大于a的键；T2修改b
	- T1访问到结点A，T2访问到结点B，B在A右侧
	- T1希望访问B，但无法获取B的读锁，因为T2持有B的写锁
- 此时T1有3种选择
	- 等待（阻塞、自旋）
		- 无法判断等待多久，一个线程不知道另一个线程在做什么
		- 可能诱发死锁，例如T2删除后B结点低于半满，希望移动A结点元素
	- 重启自己
		- 这是最优的。但不是直接重启，而是依据T1已经做出的行为等待一个合理的时间。若T1已经更新了大量内容，则重启需要回滚大量内容。
		- 可以跟踪回滚次数，若回滚过多，则可以优先调度。但这应该在高层实现而不是在B+树中实现。
	- 中断T2
		- 该选项是无法实现的。若维护一个标记，我们无法判断T2会在何时检查该标记。
- 在纳秒、微秒级别的操作上，回滚是最优的

> The leaf node sibling latch acquisition protocol must support a "no-wait" mode.
- 无等待策略 no-wait mode
	- 线程尝试获取锁失败时，不阻塞，不自旋，直接失败返回
	- 现代数据库通常可以根据实际吞吐量选择合适的策略

> The DBMS's data structures must cope with failed latch acquisitions.  
- 底层的存储允许返回获取锁失败的错误，DBMS需要能够处理这些错误
	- 通常线程在闩锁获取失败时直接返回错误，该错误对上层是透明的，上层会再次发起调用


# 9 Lecture 10: Sorting and Aggregating
---
## 9.1 Query Plan
- 输入一个SQL查询后，解析器会组织出语法树，语法树的叶子是IO指令，根结点是输出，数据自底向上流动
- 我们使用BufferPoolManager来管理内存，我们不能假定所有数据都被存放在内存中
- 希望最大化顺序读写以适应磁盘

## 9.2 Top-N Heap Sort
- 当指定获取最大（最小）的N个元素时，可以使用堆来维护
	- `ORDER BY ... LIMIT ...`
	- 虽然说是堆，但是实际需要维护最小和最大值，因此用双向链表

## 9.3 External Merge Sort
- 这里的外部排序与ADS的外部排序有所不同，数据以Page为单位而不是以条为单位
	- 假定我们使用B个Buffer Pool Page，总共有N页数据
	- Phase1：将数据分割成较小的有序的顺串（run）
		- 用B个Page读入磁盘上数据，内部排序这些数据形成顺串，写到磁盘上
		- Phase1结束时得到$\frac{N}{B}$个顺串，这些顺串都存储在磁盘上
	- Phase2：迭代的合并顺串为更大的顺串
		- B-1个Buffer Pool Page分别读入一个顺串，合并这些顺串，用剩下的一个BuffePool Page作为Output Buffer将合并的结果写到磁盘上
		- 重复上述过程，用B-1个Page作为Input Buffer读入顺串，用1个Page作为Output Buffer写出顺串
	- 上述过程是B-1路归并，我们没有考虑磁带的数量
- 顺串的内容
	- Key：被比较的键
	- Value：可以存储数据也可以存储指针，分为Early Materialization/Late Materialization
- pass数：$1+\left\lceil  \log_{B-1} \frac{N}{B}  \right\rceil$
	- 与$1+\left\lceil  \log_{k} \frac{N}{M}  \right\rceil$区分：上式中N的单位是Page，本式中N的单位是Record，k和B-1是等价的，我们未考虑磁带数
- IO Cost：$2N \cdot \text{\#pass}$
- Double Buffering Optimization
	- Buffer数：2k个input buffer，2个output buffer

### 9.3.1 Comparison Optimization
- key间的比较可能需要调用自定义的比较函数，这会导致不断的jmp
- Code Specializtion
	- 内联化
- Suffix Trunction
	- 先比较一个前缀

## 9.4 Using B+Trees for Sorting
- 在B+树中，数据的存储已经有序了，可以通过遍历叶子链来获取有序的数据
- Clustered B+Tree
	- 叶子中索引的顺序与对应的Page在磁盘上存储的顺序一致
	- 可以直接遍历叶子索引读取数据，这在磁盘上也是顺序读写，速度很快
- Unclustered B+Tree
	- 索引顺序与Page顺序不一致
	- 此时遍历叶子索引在磁盘上是随机访问
	- 对于Top-N和较小的N这是可以的


## 9.5 Aggregation
- 聚合可以使用排序也可以使用hash，两种方法各有优劣，目前而言hash略优

### 9.5.1 Sorting Aggregation
- 当已经要求排序时，在排序的基础上聚合是好的
![[Pasted image 20251228171655.png]]
### 9.5.2 Extrenal Hashing Aggregation
- Hash更优的场景
	- 无需排序，仅去重
- Hash允许操作在内存中完成，减少磁盘IO
- 当hash表太大不能存放在内存中时
	- Phase1：Partition
		- hash到无序的桶中
		- 假定有B个Buffer Pool Page，用B-1个作为分区Output Buffer，1个作为Input Buffer。
		- Phase1会把所有键映射到B-1个分区中，这些分区会被存储在磁盘上
	- Phase2：ReHash
		- 逐个引入分区，对于一组分区，用另一个hash函数建立in-memory哈希表，用这张in-memory的hash表做计算，得到的结果存储起来。一组分区计算完后，丢掉这张hash表，开始处理新一组分区。
		- 不同分区间相互独立：若两个键相同，其必然被映射到同一分区中。因此不同分区间计算的结果不会相互影响，只需要简单组合而不需要进一步复杂的聚合。
		- 引入分区的过程是磁盘上的顺序IO
		- 核心思想：在多次hash中，数据被不断切分
		- 磁盘很便宜，我们可以认为有无穷的磁盘资源
![[Pasted image 20251228171720.png]]
![[Pasted image 20251228171753.png]]