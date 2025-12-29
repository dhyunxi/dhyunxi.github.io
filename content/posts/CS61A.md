


# CS61A
---
## Function
### 函数的创建和调用
- 以python为示例
- 创建函数
	- 函数签名：包含了函数的名称、形参，告诉计算机如何创建函数栈帧
	- 函数体：包含了函数封装的流程
```python
def square(x):       # function signature
	return mul(x,x)  # function body
```
- 调用函数
	- 创建函数栈帧
	- 绑定实参到形参
	- 返回
- 纯函数与非纯函数，Pure Function
	- 纯函数：无副作用的函数

### 环境
- 环境，Environment：栈帧的序列
	- 环境是解释器用来跟踪值的含义的方式
- 栈帧，Frame：名称到值的映射的集合

- 函数在创建时不仅仅确定函数体和形参，还确定父栈帧，即环境闭包
```python
function <name> (<formal parameters>) [parent=<parent>]
```
- 当函数被调用时
	- 创建调用栈帧
	- 环境闭包会被复制到当前帧
	- 执行函数体
```python
# 例1
def f():
	print(x)
def g():
	x=1
	f()
g()	
# 调用g显示x未定义
# 这是因为f的父帧是全局环境，虽然f在g中被调用，但是不会查找到g中定义的x

# 例2
a=3
def f():
	return a+3
>> f()     6
>> a=10
>> f()     13
# 本例说明环境闭包的追踪是实时的
```


### 复合语句
- 复合语句， Compound Statement
	- 一个复合语句由多个子句（Clause）构成
	- 子句由Header和Suite构成
	- Header控制Suite的执行
```python

if x>0 :          # Header
	x=x+1         # Suite
	return -x     # Suite
else:             # Seperating Header
	x=x-1         # Suite
	return x      # Suite
```
- 条件表达式
- 迭代表达式
- 逻辑短路AND，OR

### 高阶函数
- 接受函数作为参数或返回值的函数


### lamda函数
- 和def的区别在于
	- def时自动伴随函数名称，而lamda函数没有名称
	- 区别不大

### Currying柯里化
- 柯里化：将一个多元函数转化为高阶单元函数的过程
```python
# currying 2-tuple function:
def curry2(f):
	def g(x):
		def h(y):
			return f(x, y)
		return h
	return g
	
# 以 add(x, y) == x+y 为例
# curry2(add) == lamda x: lamda y: x+y   将二元函数拆分为了两个一元函数
# curry2(add)(1) == lamda y: 1+y
# curry2(add)(1)(2) == 1+2 == 3
```
- 可以理解为多元函数逐参数填入
- 柯里化提高了可扩展性，使得每一步都被封装起来，将一个表达式拆分为了层层递进的函数
- 函数式编程

### Functional Abstraction
- 函数式抽象
	- 将过程封装为函数
	- 只知道作用而无需具体细节


### Error & Traceback
- Error
	- Syntax Error 语法错误
	- Runtime Error 运行时错误
		- Traceback 回溯：显示报错时正在做什么
	- Behavior Error 逻辑/行为错误


## Recursion
- self-call，一个有趣的例子
```python
# print的同时返回自己，可以连续调用
def print_all(arg):
	print('arg')
	return print_all

# >> print_all(1)(2)(3)
# 1
# 2
# 3

# 连续创建3个调用帧，父帧都是全局
```
- 另一个例子
```python
# 求和
def sum_all(x):
	print(x)
	def next_sum(y):
		return sum_all(x+y)
	return next_sum
	
# sum_all函数返回一个next_sum函数并打印环境值
# next_sum函数通过调用sum_all返回一个next_sum函数
```

- Recursive Function，递归函数：调用自己的函数
- Tree Recursion：树形递归，发生在多分支递归时

- 非记忆化递归可能效率低下
- 集合的划分问题 number partition

##  Sequence
- 序列：可迭代对象
- range与list不同
	- 二者都是可迭代对象
	- list(range(0,5,1))是一个由列表构造器（list constructor）构造的list
```python
elem in sequence
elem not in sequence
```
### Closure Property
- 封闭性：集合同样可以成为集合的元素
	- 列表的元素可以是列表
- 形成了层级结构（Hierarchical Structure）

### List Comprehension
- 列表推导
	- 根据现有列表生成新的列表
```python
[for x in LIST if x%25==0]
```
- Slicing，切片
	- 生成子列表的一种简写方式
	- `[1:3]`称切片操作符
	- 切片赋值会创建独立元素，对切片的修改不影响原list
	- 但是直接修改切片会修改原list，例如`l[2:]=[]`
```python
LIST[1:3] == [LSIT[i] for i in range(1,3)]
LIST[:3] == LIST[0:3]
LIST[1:]
LIST[:]
```



### Aggregation
- python提供了一些内置函数用于聚合可迭代序列
```python
sum(iteralble[, start])
# 可选参数start作为起始项，默认为0
# 在数字求和时start无意义
# 但在其他情况下，例如列表组合时，需要设置start为同类型元素，因为默认为0，0不能加list
# sum([[1,2,3], [4,5]], [])

max(iterable[, key=func])
max(a,b,c,...[, key=func])
# key是一个一元函数，设置key时，求使key最大的值
max([1,2,3,4], key = lamda x: x*(10-x))

all()
```


### Iteration
- python提供了for遍历序列
	- for不创建新的栈帧
```python
for <name> in <expression>:
	<suite>
	...
```
- for循环执行流程
	- 评估`<expression>`，需要得到一个可迭代容器
	- 遍历容器中的元素，依次绑定到`<name>`上，并执行`<suite>`
- for允许在头部解包
```python
pairs = [[1,2],[2,3],[3,3]]
for a,b in pairs:
	if a == b:
		 cnt++
```



## Data Abstraction
- 数据抽象：将数据的表示（represent）和行为（behavior）分开成两个独立的部分
	- 数据抽象通过构造器（Constructor）和选择器（Selector）定义行为（Behavior）
	- 分辨数据抽象不在于如何构建或如何实现构造器和选择器，而在于行为

- 以有理数为例
```python
# 有理数是分数，可以用2个整数构建 r = n/d
# constructor: 接受两个整数，返回一个有理数
def racial(n,d):
	...
	return r
# selector: 接受一个有理数，返回它的一部分
def numer(r):
	...
	return n
def denom(r):
	...
	return d
```
- 我们无需关心r的类型以及具体实现，就可以直接利用构造器和选择器定义行为
```python
# behavior
def multiple(r1, r2):
	return rational(numer(r1)*numer(r2), denom(r1)*denom(r2))
```
- 构造器和选择器可以有多种实现
```python
# array implementation
def rational(n, d):
	return [n,d]
def numer(r):
	return r[0]
def denom(r):
	return r[1]
	
# high-order function implementation
def rational(n, d):
	def select(part):
		if part == 'n':
			return n
		else part == 'd':
			return d
	return select
def numer(r):
	return r('n')
def demon(r):
	return r('d')
```
- 第一个实现利用了数组，第二个实现利用高阶函数闭包
- 数据的表示不影响行为，只需要选择器能返回整数值即可
- 实现了表示和行为解耦

## Tree
- 每个tree是一个llist，第一个元素为根的值，后面的元素为子树
```python
# 构造器: 接受根和子树列表
def tree(root, branches=[]):
	for branch in branches:
		assert is_tree(branch):
	return [root]+list(branches)  # 返回一个list
def is_tree(tree):
	...
# 选择器
def root(tree):
	return tree[0]
def branches(tree):
	return tree[1:]
```
- 根据构造器和解释器定义行为
```python
def is_leaf(tree):
	return not branches(tree):
	
```

## Object
- 对象由数据（属性）和行为构成，形成抽象
- 类是对象的类型，对象是类的实例
- 在python中，一切皆对象
- 对象传参和赋值是浅拷贝

### 创建
- 建立空对象
- 调用init

### Attributes
- 类属性、类方法在所有实例间共享
- 实例属性：定义在构造函数或实例方法中，独属于实例
- python允许在类外创建新实例属性
```python
class class_name:
	class_attr = ...
	def __init__(self):
		...
		self.inst_attr = ...
		
# 可以用类查看类属性  class_name.class_attr
# 也可以在实例中查看  instance.class_attr
```
- dot operator，点运算符：`<expression>.<name>`
	- 先解析`<expression>`得到对象
	- `<name>`先匹配实例属性，再匹配类属性
### Mutability
- 变异：对象的变化
- python中，函数的对象传参是浅拷贝，有副作用
- list是可变异的
- tuple是不可变异的
- 区分名称绑定更改和对象突变
	- Name Change：`x=2   x=3`
	- Object Mutation：`x=[1,2]    x.apend(1)`

### Equality and Sameness
- 在python中，对象赋值是浅拷贝
```python
a=[1,2]
b=a     # 浅拷贝，将b绑定到了a的对象
>> a==b   True
a.append(3)
>> a     [1,2,3]
>> b     [1,2,3]
>> a==b  True
```
- 上例中两个名称被绑定到同一个对象，但也有可能是对象不同内容相同
```python
a=[1,2]
b=[1,2]
>> a==b     True
b.append(3)
>> b        [1,2,3]
>> a==b     False
```
- Equality：相等，内容一致，使用` == `判断
- Sameness：相同，对象一致，使用`is`判断
- 函数的默认可变参数实际上是同一全局对象
```python
def f(l=[]):
	l.append(1)
	return len(l)

>> f()    1
>> f()    2
>> f()    3
```

### Mutable Function
- 利用可变对象，我们可以构建可变函数，关键在闭包中的变量是可变的
```python
def make_f(x):
	arg=[x]
	def f(y):
		arg[0] = arg[0]+y     # arg[0]不是一个名称，所以不创建局部变量
		return arg[0]
	return f

>> f = make_f(100)
>> f(20)    120
>> f(30)    150

# 如果采用arg=x会报错，以下面这段代码为例
arg=3
def f():
	arg=arg+3     # (*)
	return arg
# 读到(*)时，arg作为左值，解释器认为arg是局部变量，作为右值时解释器认为arg未定义
# 注：在不采用non-local或global时，解释器总是把左值绑定在本地帧

# 采用global可以实现
arg=3
def f():
	global arg
	arg = arg+3
	return arg
>> arg   3
>> f()   6
>> arg   6
```


## Container
- 容器：能存储其他对象的对象
### Iterator
- 迭代器：用于遍历对象的工具，自身也是一个对象
	- 实现了`__next()__`方法的对象都可以是迭代器
- 可迭代容器：可以返回迭代器用于遍历该容器
	- 实现了`__iter()__`方法返回迭代器
```python
iter(iterable) --> iterator  # 返回迭代器
next(iterator) --> element   # 返回当前项，并将迭代器移到下一项
```
- 迭代器是可变对象，当调用next时，迭代器变异指向下一项
- 容器长度改变时，迭代器失效
- 迭代器可以被for循环，会改变迭代器
	- 除去迭代器遍历时会被改变而容器不会外，迭代器和容器具有类似的行为，可以有`zip(list, iterator)`
- 迭代器是惰性计算的，当访问时才会计算
- `map | filter | zip`等返回的都是迭代器

### Generator
- 生成函数：包含`yield`关键字的函数
	- 生成函数通常没有`return`，只有`yield`
- 生成器：更优雅的迭代器，无需`next`和`iter`，只需要`yield`关键字
	- 生成器由生成函数返回
	- 生成器迭代一项时，会调用生成函数，并在遇到`yield`关键字时暂停生成函数，返回当前项
	- 迭代下一项时重新启动函数至下一个`yield`
	- 函数结束时返回迭代终点
```python
# 使用yield的生成函数
def gen1(x):
	yield x
	yield -x
>> g=gen1(3)
>> list(g)     > [3, -3]

# 循环yield
def gen2(x)
	for i in range(3)
		yield x*i
>> list(gen2(3))  > [0, 3, 6]

# yield from简写循环
l = [1,2,3]
def gen3(x):
	yield from l
	
# yield from简写递归
def gen4(x)
	if x>0:
		yield x
		yield from gen4(x-1)
```
- 生成表达式

## Inheritance
- 继承
- 需要注意self指实例而非类
```python
class A:
	...
	def f:
		print(self.n)

class B(A):
	...
	def __init__(self):
		self.n=2
```
## string
- repr 和 str的区别
	- repr得到python可评估的表达式，通过eval反作用
- F-string：格式化字符串

## Interface
- 一组共享的消息和规范


## Exception
- 异常处理
```python
raise <exprission>
# <expression>需要是BaseError子类

try:
	<try suite>
except <expression> as <alias>: 
	<except suite>
final:
```
## Tail Recursion
- Tail Call：尾调用
- 尾递归可以压缩到O1空间复杂度


## SQL
```sql
SELECT [columns] as [alias]
FROM [tables]
WHERE  [conditions]
ORDER BY [order]
```
- 子句内部可以用逗号隔开形成列表
- 对于`columns`和`tables`我们可以取别名
```SQL
SELECT a.child as ac, b.child as bc
FROM TableA as a, TableB as b
WHERE a.parent = b.parent
```
- 实际上用逗号隔开的列表`[tables]: TableA, TableB`会通过笛卡尔积形成一个新表。再通过`where`选取特定行，再通过`columns`选取特定列
- 用`||`连接字符串
- Aggression：在`[columns]`中添加行聚合
	- `sum avg min max count`
```sql
GROUP BY <expression> having <expression>
```
- 分组后，聚合按组生效
# CS61B
---
