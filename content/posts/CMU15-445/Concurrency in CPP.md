
# 1 Thread
---
## 1.1 thread类
- 构造函数
	- 接收一个可调用对象
	- 线程在被构造完后立即开始执行
	- 每个线程只能调用一次，调用完毕后设为`joinable = false`
- 执行模式
	- `T.join()`：主线程将等待子线程
	- `T.detach()`：主线程不等待子线程
	- `T.joinable()`：子线程能否join
- 什么都不加代表主线程也不会考虑子线程
- 主线程结束时强制结束子线程
```cpp
auto Func(auto args) -> auto {}

// 用线程封装函数
std::thread T( Func(params) );

// 匿名线程
std::thread (Func, [params]);

// 前台启动线程
T.join();
// 后台启动线程
T.detach();
```

## 1.2 this_thread
- 命名空间
```cpp
namespace std::this_thread{
	get_id() // 获取线程id
	yield() // 立即让出CPU，可以被重新调度
	sleep_for() // 睡眠
	sleep_until() // 睡眠
}
```

# 2 Mutex
---
- 互斥锁
	- `mutex`
	- `recursive_mutex`
	- `time_mutex`
	- `recursive_timed_mutex`
```cpp
// 为函数上锁
#include <mutex>
std:: mutex mtx;
void Print(char c){
    mtx.lock();
    for (int i=0; i<n; ++i) 
    {
       std::cout << c; 
    }
    std::cout << '\n';
    mtx.unlock();    
}

int main ()
{
    std::thread th1 (print_block,50,'');//线程1：打印*
    std::thread th2 (print_block,50,'$');//线程2：打印$

    th1.join();
    th2.join();
    return 0;
}
// 按顺序打印
// ************
// $$$$$$$$$$$$

mtx.lock() // 上锁
mtx.unlock() // 解锁
mtx.trylock() // 若被其他线程上锁，True； 若未上锁，返回false并锁住
// 若已经被同线程上锁，死锁
```
- 需使用同一互斥锁实例


## 2.1 Deadlock
- 两个线程相互等
	- T1先m1，再m2；T2先m2，再m1
	- 某个时刻T1获得m1，T2获得m2，死锁
- 同一线程连续获取同一锁
```cpp
std::mutex mtx;
void foo(){
	mtx.lock();  // 获取到锁
	mtx.lock();  // 阻塞
	...
}
```

## 2.2 lock_guard
- 模板类
- 自动锁：
	- 调用构造函数时加锁，调用析构函数时解锁
	- 通常作为局部变量，以生命周期充当锁周期
	- 会阻塞
```cpp
int shared_data;
std::mutex mtx;

void foo(){
	std::lock_guard<std::mutex> lg(mtx); // 实例化时加锁
	...
}   // 函数结束，局部变量析构，解锁

// 等效于
void foo(){
	mtx.lock();
	...
	mtx.unlock();
}

// 可以传入adopt_lock表示已经提前上锁，会在析构时解锁
void foo(){
	std::lock_guard<std::mutex> lg(mtx, std::adopt_lock);
}
```
- 禁用移动构造和拷贝构造

## 2.3 Unique Lock
- 自动锁：
	- 与`lock_guard`相同
	- 也可以传入`std::adopt_lock`表示已经上锁
- 尝试锁：
	- 尝试获取锁，失败返回false而非阻塞
	- 两种实现
		- 传参`std::try_lock`使获取锁失败也不阻塞
		- 传参`std::defer_lock`，再调用`try_lock_for()`
```cpp
// 非阻塞锁有两种实现

// 第一种：
//   在构造时尝试加锁，失败也不阻塞
//   通过unique_lock.owns_lock()判断是否持有锁
std::mutex mtx;
void foo(){
	std::unique_lock<std::mutex> ul(mtx, std::try_to_lock);
	if(ul.owns_lock()){
		// 持有锁
		...
	}else{
		// 不持有锁
		...
	}
}
// 同一线程在已经持有锁时再次try_to_lock同一锁会失败
// 不会直接死锁，但会逻辑混乱
mtx.lock()
unique_lock<mutex> ul(mtx, try_to_lock);
// ul.owns_lock() == false


// 第二种：
//   传参std::defer_lock不会自动加锁且不持有锁，但如果手动加锁，会设置持有锁
//   try_lock_for方法：尝试获取锁一段时间，若超时返回false，成功返回true
//   这里锁的类型需要为时间锁
//   在构造前加锁会编译错误
std::timed_mutex mtx;
void foo(){
	std::unique_lock<std::timed_mutex> ul(mtx, std::defer_lock);
	if(ul.try_lock_for(std::chrono::seconds(5))){
		 ...
	}
	...
}

// unique_lock 也支持直接lock和unlock
```
- 实现逻辑：
	- `unique_lock`有两个重要属性
		- `mtx`：目标锁
		- `owns`：是否持有目标锁
	- 析构函数中
		-  当且仅当`owns == true`时解锁
	- 只传入锁`mtx`的构造函数
		- 设置`owns = true`
		- 加锁
	- 传入锁`mtx`和`std::adopt_lock`的构造函数
		- 设置`owns = true`
		- 不加锁
	- 传入锁`mtx`和`std::defer_lock`的构造函数
		- 设置`owns = flase`
		- 不加锁
	- 传入锁`mtx`和`std::try_lock`的构造函数
		- 设置`owns = false`
		- 不加锁
	- 手动加锁，即调用`ul.lock() | ul.try_lock_for() | ul.try_lock()`
		- 若成功
			- 设置`owns = true`
			- 加锁
		- 注意第一个会阻塞，后两者不阻塞



## 2.4 recursive mutex
- 递归锁

## 2.5 call once
- 用`std::once_flag`标识是否调用


## 2.6 condition variable
- 类
	- `condition_variable`：必须结合`unique_lock`使用
	- `condition_variable`：可以结合任意锁使用
- 阻塞当前线程直到被通知
```cpp
// methods
wait() // wait until notified
wait_for() // wait for timeout or notified
wait_until() // wait for timepoint or notifiled
notify_one() // notify one thread
notify_all() // notify all thread
```
- `void wait(std::unique_lock<std::mutex>& lock, Predicate pred);`
	- 第二个参数需要是谓词类型
	- 调用wait前线程已经获得锁
	- 调用wait时，执行pred
		- pred返回false时，wait会阻塞当前线程，并通过unlock释放锁，让别的线程能够使用资源
		- 返回true时，继续执行
	- 收到notify时，执行pred
		- 返回true时，调用lock获取锁
		- 返回false时，继续阻塞
```cpp
std::mutex mtx;
std::condition_variable cv;
std::queue tasks;

bool has_task(){
	return !tasks.empty();
}
void foo()
{
	std::unique_lock<std::mutex> lock(mtx); // 获得锁
	cv.wait(lock, has_task);
	// 传入lock
	// 调用has_task(), 若返回false：
	//           阻塞当前线程, 释放锁lock.unlock()
	// 接收到notify通知
	// 调用lock.lock()重新获得锁, 停止阻塞
	task_t task = tasks.front();
	tasks.pop();
	lock.unlock();
	task();
}
```

## 2.7 线程池
- 组成部分
	- 线程组
	- 任务队列：共享数据，需要锁
	- 添加任务接口
	- 线程池管理器：创建并管理线程
- 线程复用
	- 线程在执行完毕后自行销毁，不能join两次
	- 线程的创造和销毁很占用资源
	- 在线程内设置永久循环，用条件变量阻塞，实现复用


# 3 异步
---
- 在异步任务中，线程会把一个任务传递给另一个线程执行，并获取执行的结果。
- 在`<future>`库中，将异步任务角色分为Provider和Receiver
	- Provider：提供结果（**而非任务**）的一方
		- 创建和管理共享状态
		- 设置计算结果
		- 通知等待的线程
	- Consumer：需求结果的一方
		- 获取计算结果
		- 在结果未生成前阻塞
		- 通常由Provider生成
	- 共享状态：在Provider和Consumer间共享的数据
		- 数据是Provider -> Consumer单向流动的
		- 共享状态只能设置一次


## 3.1 future
- 异步任务Consumer，通常由Provider生成，但也有自己的构造函数
- Consumer可分为独占式和共享式
	- `std:future<>`：独占一个Provider
		- `future`不支持拷贝构造，只能移动构造，在传递`future`时共享状态被转移
	- `std::shared_future<>`：可以与其它Consumer共享Provider
		- `shared_future`支持拷贝构造，在拷贝时会共享同一个共享状态，从而实现多线程共享
	
- `future`对象可以由三种方式创建
	- `future`本身的构造函数
	- Provider类：
		- `std::promise<>`
		- `std::packaged_task<>`
	- Provider函数：
		- `std::async()`

### 3.1.1 构造函数
- future
	- 禁用拷贝构造、拷贝赋值
	- 允许移动构造
- `shared_future`
	- 允许拷贝，允许移动
	- 可以从`future`隐式转换
```cpp
int foo();
std::future<int> future_result(std::launch::async, foo);
// 相当于创建了一个foo的线程

future_result.get();
```
### 3.1.2 成员函数
- `get()`：Consumer核心功能，获取共享值
- `valid()`：返回future对象是否可用
- `share()`：返回`shared_future`对象，且返回后自身不在可用
- `wait()`：等待共享状态ready，但不读取共享状态
- `wait_for()`：等待ready，超时后返回
	- `future_status::ready`：ready
	- `future_status::timeout`：超时
	- `future_status::defered`
- `wait_until()`


### 3.1.3 相关函数
#### 3.1.3.1 std::async()
- 返回一个future


### 3.1.4 相关枚举
- 



## 3.2 promise
- 用于线程间通信
- 与future绑定使用
- 期望子线程产生一个值，并在本线程中获取此值
```cpp

void foo(std::promise<int>& p){
	p.set_value(100);
}

int main()  // 主线程
{
	std::promise<int> p;
	auto future_result = p.get_future();  // 期望获得的值
	
	std::thread T(foo, std::ref(p));
	T.join();
	
	std::cout<<future_result.get();
	return 0;
}
```

- 可以作为Provider返回一个future
	- 返回的future与promise有相同的共享状态
- 通常将生成的future传递给其它线程，然后用promise设置共享状态
- 构造函数
	- 禁用拷贝构造、拷贝赋值
	- 允许移动构造
- 可以使用`set_expection()`设置共享状态为异常
```cpp

void foo(std::future<int>& fut){
	int result = fut.get();
	std::cout<<result<<std::endl;
}

int main()  // 主线程
{
	std::promise<int> p;
	auto future_result = p.get_future();  // 期望获得的值
	
	std::thread T(foo, future_result);
	T.join();
	
	p.set_value(100);
	return 0;
}
```
-  std::promise::set_value_at_thread_exit
设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready。如果某个 std::future 对象与该 promise 对象的共享状态相关联，并且该 future 正在调用 get，则调用 get 的线程会被阻塞，当线程退出时，调用 future::get 的线程解除阻塞，同时 get 返回 set_value_at_thread_exit 所设置的值。注意，该函数已经设置了 promise 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 future_error( promise_already_satisfied )。



## 3.3 共享状态生命周期
- Provider消亡后，future依然可以访问共享状态



## 3.4 Atomic
- 封装，使并行读写是良定义的
- 禁用拷贝和移动
- methods
	- `is_lock_free`：
	- `store`：写
	- `load`：读
	- `exchange`：写入新值并返回旧值
	- `wait()`：阻塞直到被通知或值改变
	- `notify_one`：通知一个
	- `notify_all`：通知所有


# 4 关于引用的问题
- 在函数模板中，右值引用是万能引用
	- 万能引用会把左值推导为左值引用
- 左值引用形参
	- 接收左值、左值引用
- 右值引用形参
	- 接收右值、右值引用、左值引用
	- 右值引用绑定到右值时，右值内容的资源转移到了右值引用形参下，右值触发析构，但右值释放时，资源仍然存在于形参名下
	- 右值引用本身无意义，意义在于触发移动构造
- 触发隐式转换时会创建临时对象，该临时对象可以被右值引用接收




# 其它
- 线程在被构造时立即启动
- T.join()代表主线程将等待子线程完成后再继续运行
- T.detach()代表主线程不会考虑子线程
- 什么都不加代表主线程也不会考虑子线程
- cpp要求线程对象在析构时不是joinable状态，否则会报错`terminate called without an active exception`，调用std::terminate()
- 主线程结束时强制结束子线程