
- 本章以Java的JUC库为例讲解锁机制


# 1 锁机制
---
- Java使用`syncronized`实现线程间同步
	- `syncronized`会创建内置锁（Intrinsic Lock）或管程（Monitor）
	- 管程
		- 管程有执行区和等待区
		- 执行区只能同时容纳一个线程，线程运行完挂起时，管程会通知等待区中的一个线程进入执行区开始执行
- 线程的挂起和执行切换是os层面的，很耗时间
- Java6优化了`syncronized`，拓展了锁，从低到高为
	- 无锁
	- 偏向锁
		- 锁记录了一个线程ID，偏向这个线程。通常偏向的是第一个获取该锁的线程
		- 当有其它线程来获取偏向锁时，升级为轻量锁
	- 轻量锁
		- 线程通过CAS尝试获取锁
		- 持有锁的线程在Lock Record中拷贝锁的MarkWord，并用owner指针指向锁
		- 锁记录指向持有锁的线程的Lock Record的指针
		- 轻量锁是自旋等待的
		- 当有超过1个线程自旋时，升级为重量锁
	- 重量锁
		- 通过os的mutex实现
		- 重量锁是阻塞的，需要os调度
	- 上述过程也称为锁的升级或锁的膨胀
- notions
	- 自旋锁 Spin Lock
		- 尝试获取锁，阻塞时不挂起，而是轮询，此时CPU空转，长时间自旋会浪费CPU资源
	- 忙等待
		- 线程的自旋又称为忙等待
	- 阻塞锁、互斥锁 Blocking Lock、Mutex
		- 获取锁失败的线程会被挂起，进入等待队列
	- 非阻塞锁、尝试锁
		- 获取锁失败则直接返回false，不进入等待队列
	- 公平锁
		- 先请求先获得
	- 非公平锁
		- 与公平锁相对
	- 可重入
		- 允许一个线程在同一上下文中重复获取同一把锁
		- 若不允许重入，则当线程持有锁时，若再次尝试获取锁，会进入死锁


# 2 乐观锁
---
- 悲观锁 Pessimistic Concurrency Control
	- 必须限制资源只能被一个线程访问
- 乐观锁 Optimistic Concurrency Control
	- 总是认为资源不被占用

## 2.1 CAS
- Compare and Swap
	- 线程持有一个旧值old value与新值new value
	- 线程会对比资源的当前状态，若与旧值相同，则将其覆写为新值
- 操作系统提供了原子性的CAS
	- 本质是CPU提供了锁，从而避免了上层锁


## 2.2 Atomic Class
- Java中的原子类底层通过CAS实现
	- AtomicInteger


# 3 AQS
- Abstract Queued Synchronizer
- JUC中的抽象类，该类是众多锁的基础
## 3.1 属性
- `private volatile int state`：对象的同步状态
	- int而非bool是为了支持共享锁
	- 该属性的值的具体含义由子类定义
- `head tail`：等待队列的头尾指针
	- 等待队列是双向链表，每个结点封装一个等待线程
	- 头结点是哨兵结点，存储线程但无意义
## 3.2 方法
### 3.2.1 获取资源
- `tryAcquire`：尝试获取资源
	- 抽象方法，AQS不提供默认实现
	- 尝试获取锁一次，成功时返回true，失败时返回false
```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```
- `Acquire`：尝试获取资源
	- 模板方法，调用`tryAcquire`
	- 尝试获取锁，失败时不放弃，进入等待队列
```java
public final void acquire(int arg) {
    // 1. 先尝试快速获取
    if (!tryAcquire(arg) &&
        // 2. 失败后加入等待队列
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        // 3. 如果被中断过，恢复中断状态
        selfInterrupt();
}
```
### 3.2.2 入队
- `addWaiter`：封装线程为结点，加入等待队列
	- 若队列不为空
		- 尝试一次快速入队
	- 若队列为空或CAS失败
		- 进入完整入队`enq`
	- 只有CAS成功的结点会修改旧尾的后继，因此是线程安全的
- `enq`：持续尝试入队
	- 自旋
```java
private Node addWaiter(Node mode){
	Node node = new Node(thread, mode); // 封装线程为结点
	Node pred = tail;  // 获取尾结点
	if(pred != null){
		node.prev = pred;
		if(compareAndSetTail(pred, node)){   // CAS修改尾结点
			pred.next = node;                // CAS成功，修改旧尾的后继
			return node;
		}
	}
	enq(node);  // 调用自旋入队
	return node;
}

private Node enq(final Node node){
	for(;;){  // 自旋
		Node t = tail;
		if(t == null){
			if(compareAndSetHead(new Node())){
				tail = head;
			}
		}else{
			node.prev = t;
			if(compareAndSetTail(t, node)){
				t.next = node;
				return t;
			}
		}
	}
}
```
- `acquireQueued`：队列中的结点请求获取资源
	- 自旋
		- 若排在第一位（前驱是头结点），尝试获取资源
		- 获取失败，判断是否需要挂起
			- 该过程会同时清理队列中被取消的线程结点
	- 被挂起
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;  // 标记是否获取失败
    try {
        boolean interrupted = false;  // 标记是否被中断过
        for (;;) {  // 自旋
            final Node p = node.predecessor();  // 获取前驱节点
            // 条件1：前驱是头节点（当前是队列第一个等待者）
            // 条件2：尝试获取资源
            if (p == head && tryAcquire(arg)) {
                setHead(node);      // 获取成功，设置为头节点
                p.next = null;      // 断开原头节点，帮助GC
                failed = false;     // 获取成功
                return interrupted; // 返回中断状态
            }
            
            // 获取失败，判断是否需要挂起线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;  // 记录中断状态
        }
    } finally {
        // 只有发生异常（如取消）才会进入这里
        if (failed)
            cancelAcquire(node);  // 取消节点获取
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;  // 前驱节点的等待状态
    if (ws == Node.SIGNAL)     // -1，表示前驱节点释放时会通知我
        return true;           // 可以安心挂起
    
    if (ws > 0) {  // 前驱节点已取消（CANCELLED = 1）
        // 跳过所有取消的前驱节点
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {  // 0 或 PROPAGATE(-3)
        // 将前驱节点状态设为 SIGNAL，确保会通知我
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;  // 这次先不挂起，再试一次
}
```
### 3.2.3 释放
- `tryRelease`：尝试释放结点
	- 抽象方法
- `release`：释放锁
	- 模板方法
	- 锁掌握在头结点手里，当释放时，意味着头结点完成了任务，成为了虚结点
	- 将头结点的等待状态置为默认值0，然后唤醒其后继
- `unparkSuccessor`：唤醒结点的后继
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {          // 1. 尝试释放
        Node h = head;              // 2. 获取头节点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);     // 3. 唤醒后继节点
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)  // 如果状态为SIGNAL/PROPAGATE等
        compareAndSetWaitStatus(node, ws, 0);  // 清零
    
    Node s = node.next;  // 后继节点
    if (s == null || s.waitStatus > 0) {  // 后继为空或已取消
        s = null;
        // 从尾向前遍历，找到有效的后继节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);  // 唤醒线程
}
```


# 4 ReentrantLock
基于AQS，支持公平锁和非公平锁，支持可重入，调度更灵活
re-entrant-lock：重-入-锁
- ReentrantLock是独占锁
- 相较synchronized
	- 可中断
	- 公平锁
	- 延迟锁
	- 多条件变量
- Java6后Synchronized得到优化，现在性能已经差不多了

## 4.1 Lock
- ReentrantLock实现了Lock接口
	- lock是区别于synchronized的另一种具有更多广泛操作的同步结构，能支持更多灵活的操作，关联更多的条件变量，支持延迟锁
	- `tryLock`：尝试一次获取锁，失败返回false
	- `lock`：获取锁，失败时进入等待队列
	- `lockInterruptibly`：可中断加锁
	- `unlock`：解锁
	- `newCondition`：注册新的条件变量
- `lock`和`tryLock`实际是对`acquire`和`tryAcquire`的封装


## 4.2 Sync类
- ReentrantLock内部实现的抽象类
	- 继承AQS
	- 拥有子类FairSync和NonfairSync，实现了公平锁和非公平锁
- `lock`
	- 空实现，抽象方法
- `nonfairTryAcquire`
	- 非公平获取
	- 先获取锁状态
		- 当锁状态空闲时，尝试获取锁
			- 成功返回true，失败返回false
		- 锁状态不空闲，判断是否为重入
			- 重入则修改锁状态，返回true
			- 非重入返回false
	- 该方法在`Sync`中实现而非`UnfairSync`中实现的原因是在`unlock`中调用`nonfairTryAcquire`
- `tryRelease`
	- 释放锁
	- 若锁被**完全**释放（重入次数清零），返回true，否则返回false

## 4.3 NonfairSync
- 实现了非公平锁
	- 继承Sync
- `lock`
	- 一次CAS尝试获取锁，获取失败调用`acquire`
		- `acquire`是AQS的模板方法，会调用`tryAcquire`再尝试一次获取锁，失败则进入等待队列
	- 这里的CAS是非公平的体现，允许线程插队
	- 若CAS失败，则退化为公平锁
- `tryAcquire`
	- 实现AQS的抽象方法
	- 调用`nonfairTryAcquire`

- 这里对获取锁`lock()`的流程做一个分析
- 锁状态
	- 空闲
		- CAS获取锁成功，返回true
	- 自己持有锁
		- CAS获取锁失败，调用`acquire`
			- `acquire`调用`tryAcquire`
				- 重入成功，返回true
	- 别人持有锁
		- CAS获取锁失败，调用`acquire`
			- `acquire`调用`tryAcquire`
				- 获取失败，返回false
			- 进入等待队列

## 4.4 FairSync
- 实现了公平锁
	- 继承Sync
- `lock`
	- 直接调用`acquire`
	- 与非公平锁相比没有CAS的插队
- `tryAcquire`
	- 获取锁状态
		- 空闲：判断自己是否是队列第一位
			- 是第一位则CAS获取锁，返回true
			- 不是第一位则返回false
		- 非空闲：判断是否是重入
			- 是重入则修改锁状态，返回true
			- 不是则返回false


## 4.5 ReentrantLock
- `sync`
	- 成员变量
	- 公平锁或非公平锁
- `lock()`
	- 调用`sync.lock()`
- `tryLock()`
	- 调用`nonfairTryAcquire()`
- `unlock()`
	- 调用`sync.release()`

# 5 Countdown Latch