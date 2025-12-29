
实现内存-硬盘的IO管理，提供将硬盘抽象为内存的服务

- 接收请求
	- 若内存命中，则返回指针
	- 内存未命中，从硬盘中读取页到缓存池
- 调度请求任务队列
- 替换策略：LRU-K维护缓存池热点数据


# 1 LRU-K
---
- 提供一个回收站
	- 不再使用的frame将被移入回收站，但未被删除
	- 当需要空间时，从回收站删除最没用的
## 1.1 Policy
- 记录
	- 最近K次访问的时间戳
	- 最近一次访问距离最近第K次访问的时间距离，若最近访问少于K次，则+INF
- 驱逐
	- 驱逐K-时间距离最大的帧
	- 若存在相同的时间距离，则选择最近一次访问时间最晚的帧
- 容量
	- Replacer的最大容量与Buffer Pool Manager一致
	- Replacer的容量是其中可驱逐帧的个数
	- 当且仅当可驱逐帧加入Replacer或不可驱逐帧被标记为可驱逐时，Replacer的容量增加
- Node
	- history_：记录历史
	- k_：历史个数
	- fid_：对应的frameid
	- is_evictable：是否可驱逐
- 用是否出现在`*_nodes`中表示是否可驱逐，无需维护node中的`is_evictable`，由Replacer维护


## 1.2 Implementation
### 1.2.1 Attr
- Replacer
	- `node_store_`：frameID到node的映射
		- `Evict`会驱逐节点
		- `Remove`会驱逐节点
		- `RecordAccess`更改节点history
	- `cold_nodes_`：访问次数（小于K）到frameID
	- `hot_nodes_`：访问次数（等于K）到frameID
	- `current_timestamp_`
	- `curr_size_`
- node
	- `Last`：最前一次访问的时间戳
	- `isHot`：访问次数是否达到K次
	- `node_access`：访问当前结点
	- `is_evictable`：是否可以被驱逐
- 无需处理INF，用cold和hot区分
- 不可驱逐的frame会被保存在`node_store_`中，但不会被保存在`*_nodes_`中
- 访问frame时，将对应的node设为不可驱逐，并
### 1.2.2 Evict
- 有冷节点淘汰冷节点最后一个
- 没有冷节点淘汰热节点
- 若成功淘汰，减少Replacer大小
- 注意
	- 脏页写回需要在上层考虑
### 1.2.3 RecordAccess
- 访问的节点一定变成不可驱逐
- 如果不在buffer中
	- 创建结点
- 更新历史
- 更新时间戳
- 注意该函数无需修改isEvictable属性，这由`SetEvictable`处理

### 1.2.4 SetEvictable
- 若frameID不合法，抛出异常
- 根据frameID取得node
- 若node的isEvictable与参数一致，则返回
- 修改`node.is_evictable`
- 设为可驱逐
	- 根据`isHot`判断插入冷队列还是热队列
- 设为不可驱逐
	- 根据`ishot`从队列中移除
- 修改`curr_size_`

### 1.2.5 Remove
- 根据`frame_id`移除一个可驱逐frame



# 2 Disk Scheduler
---
- 调度发送给DiskManager的请求
	- 接受DiskRequest，维护Channel（等待队列），具体的任务发送给Disk Manager执行
	- 有一个持续取任务并发送给manager的工作线程
- 在Channel类中实现线程安全，IO无需再加锁
- **任务交给manager后就设置callback为true？？？**
## 2.1 Attr
- `*disk_manager_`：指向Disk Manager的指针，在构造函数中初始化
- `Channel<optional<DiskRequest>> request_queue_`：任务队列
	- 空任务表示Scheduler析构
- `optional<thread> background_thread_`：后台消费者线程
	- 在Disk Scheduler中被创建
	- Scheduler析构时join

## 2.2 Method
- schedule
	- 调度函数
	- 
### 2.2.1 构造函数
- 参数
	- `DickManager* disk_manager`
- 初始化disk manager指针
### 2.2.2 析构函数
- 将`std::nullopt`加入任务队列，标识后台工作线程退出
- 将`background_thread_`join


### 2.2.3 Schedule
- 添加任务
- `void Schedule(std::vector<DiskRequest> &requests)`
	- 取第一个元素
		- 移动构造optional
	- 移除requests第一个元素
	- optional加入任务队列
	- 循环直到空


### 2.2.4 Create Promise
- 创建Promise

### 2.2.5 Deallocate Page
- 调用manager的DeletePage

### 2.2.6 Start Worker Thread
- 介绍
	- 线程`background_thread_`执行的函数，当DiskScheduler析构时返回
	- 需要调用manager的方法执行任务
	- 执行完设置promise为true
- `void StartWorkerThread()`
	- 循环
		- 从任务队列中提取任务
		- 调用manager执行任务
		- 设置任务的`callback_`为True
	- 遇到`std::nullopt`时，循环结束，返回
	- 注意
		- Channel实现了取任务阻塞


## 2.3 Class：Channel
- Channel类是实现了线程安全的任务队列
- 支持多生产者和多消费者
### 2.3.1 Attr
- `m_`：任务队列锁
- `cv_`：条件变量
	- 在自己的get和put方法中使用
- `queue_`：任务队列

### 2.3.2 Method
- `void Put(T element)`
	- `unique_lock`上锁
	- 任务加入队列
	- 解锁
	- 条件通知所有
- `auto Get() -> T`
	- `unique_lock`上锁
	- 条件等待
	- 取任务
	- 返回任务

## 2.4 Class: DiskRequest
- 代表了一个DiskIO请求
- Attr
	- `bool is_write_`：是否是写请求
	- `char* data_`：要写入的数据、读出的数据存储位置
	- `page_d_t page_id_`：访问的页ID
	- `promise<bool> callback_`：同步原语，当请求被处理时设置


## 2.5 Extra：Test
- 了解diskscheduler的test可以更好的理解scheduler的工作
- 建立manager和scheduler
	- 创建Disk Manager智能指针dm
	- 从dm返回disk scheduler
- 创建Disk Request
	- 从调用Scheduler的`CreatePromise()`得到promise
	- promise返回future
	- 构造写请求r1
		- promise1移动入r1
	- 构造读请求r2
	- 把request塞入requests向量
	- 调用scheduler的`Schedule()`方法，传入requests
- 确认任务完成
	- 用`ASSERT_TRUE`宏确认promise的future是否为true
- 结束
	- 释放scheduler
	- 析构manager


# 3 Buffer Pool Manager
---
- 功能
	- 读：从硬盘读取页面，存储在内存池中
	- 写：当被显式调用或内存池满时，将内存池中的脏页写入硬盘
- 注意
	- 使用LRU-K Replacer执行替换策略
	- 使用DiskScheduler调度硬盘IO
		- BufferPool Manager -> Disk Scheduler -> Disk Manager
	- Buffer Pool Manager不需要解析page的内容，只需要根据pageID读取page
- 实现
	- 由一个数组存储frame
	- 由一个hash表维护pageID到frameID的映射
- 并发
	- 保证同一时间一个磁盘页至多在内存中有一个帧
	- 用一把大锁锁住整个bufferpool




## 3.1 Class: Frame Header
- 辅助类
	- 维护一个frame
	- Buffer Pool Manager内部存储多个Frame Header
	- 所有frame的IO都要通过frame header
- Friend Class：`GetData`是private方法，需要友元类
	- Buffer Pool Manager
	- Read Page Guard
	- Write Page Guard
- Attr
	- `frame_id_t frame_id_`：帧ID
	- `bool is_dirty_`：脏页标记
	- `shared_mutex rwlatch_`：共享锁
	- `atomic<size_t> pin_count_`：pin的次数
	- `vector<char> data_`：数据
	- `page_id_`：维护的pageID，初始化为`INVALID_PAGE_ID`
- Method
	- 构造函数：传入frameID
	- `GetData()`：返回const数据指针
	- `GetDataMut()`：返回数据指针
	- `Reset()`：重置Header
		- 填充NULL
		- pin设为0
		- 去除脏标

## 3.2 Attr
- `num_frames_`：frame数量，需指定
- `next_page_`：维护下一个申请的空page的ID
- `shared_ptr<mutex> bmp_latch_`：buffer pool manager锁
	- 该锁保护的是`free_frame_`和`replacer_`
- `vector<shared_ptr<FrameHeader>> frames`：向量中的每个元素是指向FrameHeader的智能指针
	- BufferPool在实际应用是一串连续的内存区域，被划分为多个frame
	- 由于cpp没有内存安全，容易越界访问，因此这里以指针数组的形式存储
	- frame在向量中的offset就是frameID
- `page_table_`：从pageID映射到frameID的hash表
- `list<frame_id_t> free_frame_`：存储空frame的ID
- `replacer_`：替换策略
- `scheduler_`：指向DiskScheduler



## 3.3 Method
- 构造函数
	- 初始化BFM参数
	- 申请`frames_`、`page_table_`空间
	- 创建`num_frames_`个帧，存储在`frames`中，并全部添加到`free_frames_`中
### 3.3.1 NewPage
- Signature
	- `auto NewPage() -> page_id_t` 
- Function
	- 创建一个新的page，返回对应的PageID
- Implementation
	- 取得BFM锁
	- 取得`next_page_id`并增加`next_page_`
	- 获取空frame
		- 若有空frame：直接取得id
		- 若无空frame：
			- 驱逐一个frame
				- 无法驱逐：返回`INVALID_FRAME_ID`
				- 可以驱逐：写回脏页，删除frameID-pageID
	- 重置FrameHeader，添加脏标，记录pageID
	- 更新pageID - frameID映射
	- 返回pageID
- Note
	- 虽然说是在硬盘上申请page，但实际上只需要在memory中建立frame
	- 对应的page会在frame被写入memory时创建，我们无需关心这会在什么时候发生

### 3.3.2 DeletePage
- `DeletePage(page_id_t page_id) -> bool`
	- 删除一个Page，both in buffer pool and disk
	- 返回false若无法成功删除
- Implementation
	- 在缓冲池中查找PageID对应的Frame
		- 若找到了，尝试删除，不成功返回false
		- 修改`free_frames_`
	- 在磁盘上删除Page

### 3.3.3 Checked Write Page
- `CheckedWritePage(page_id_t page_id, AccessType access_type) -> std::optional<WritePageGuard>`
	- 为对应的Page创建WritePageGuard并返回
	- 每个Page应当只能有一个WritePageGuard，若对应的Page已经有WritePageGuard，则返回`std::optnull`
- Implementation
	- 注意构造WritePageGuard前释放bpm锁
	- 在缓冲池中查找Page
		- 命中
			- 没有pin：创建WritePageGuard并返回
			- 被pin了：返回optnull
		- 未命中，尝试驱逐
			- 不能驱逐：返回optnull
			- 可以驱逐：
				- 检查脏标，写回脏页
				- 驱逐
				- 读入Page



### 3.3.4 Flush
- `FlushPageUnsafe`
	- 直接写回脏页
- `FlushPage`
	- 先获取锁，在写回
- 之所以需要两个，是因为已经有PageGuard或BPMLatch时，不需要再获取页锁

# 4 PageGuard
- RAII对象，提供线程安全的page IO
	- 当我们想要访问一个page时，创建一个PageGuard对象来代表该Page
	- 通过将BufferPoolManager声明为友元类，构造函数声明为private，只允许BPM创建该对象
	- PageGuard
- 一些共同属性和方法的说明
	- `page_id_`：pageID
	- `frame_`：存储真正的数据指针
	- `replacer_`：用于在构造时pin，析构时setEvictable
	- `bpm_latch_`：bpm锁
	- `disk_scheduler`：用于flush
	- `is_valid_`：默认设为false，采用非空构造时设为true。这一项存在是因为我们支持空构造。
	- PageGuard非空构造时
		- 初始化属性
		- 获取bpm锁
		- 获取frame独占锁
		- 在`replacer_`中锁定frame
		- 增加frame的`pin_count`
	- PageGuard析构时
		- 减少frame`pin_count`
		- 释放frame独占锁
		- 获取bpm锁
		- 在`replacer_`中释放frame
## 4.1 WritePageGuard
- 同一时间一个Page只能有一个WritePageGuard，且此时没有ReadPageGuard
- 构造函数
	- 空构造
		- 不应使用默认构造的WritePageGuard，默认构造的意义在于用于后续移动构造
	- 主要构造
		- 获取bpm锁
			- 注意在这之前需要释放bpm锁，否则会死锁
		- 该构造需要做获取Page写权限的所有事情
			- 设置脏标
			- 增加frame的pincount
			- 在replacer中pin该frame
	- 移动构造
		- 先判断对方的`is_valid_`
- `Flush`
	- 调用`disk_scheduler_`
- `Drop`
	- `is_valid_==false`则什么都不做


# 5 Disk Manager
---
- 负责硬盘上的操作
	- 硬盘读写
	- page的分配与释放
- Disk Abstraction
	- 提供逻辑文件系统抽象层
	- 外部只需要通过pageID访问page
- Disk Model
	- 构造Disk Manager时，传入`file_path`指定DB在硬盘上的存储位置
	- 空间被分割为一个个固定大小的slot
	- 懒分配：当且仅当访问新page时，为该page分配slot
	- 维护从pageID到对应slot的offset
	- 维护空slot的位置
- 利用cpp的`<filesystem>`库实现硬盘操作
## 5.1 Methods
### 5.1.1 AllocatePage
- 函数签名
	- `auto DiskManager::AllocatePage() -> size_t`
- 功能
	- 申请一个新的page
- 实现
	- 检查空slot
		- 如果有空slot：在该位置创建page
		- 如果无空slot：将page添加到文件末尾，如果需要可以申请硬盘上的更多空间
	- 返回offset
- 注意
	- 本函数无需pageID参数
	- 本函数不修改`pages_`表，需要在上层添加pageID - offset映射




# 6 Others
---
- `optional.value_or(defalut_value)`：如果有值则返回value，否则返回提供的默认值
- `vector.data()`：返回指向内部数据的指针
- `map.at()`：根据键查找值
	- 返回值的引用，或抛出out of memory
- `map.find()`：根据键查找结点
	- 返回结点迭代器
	- 不存在时返回end()