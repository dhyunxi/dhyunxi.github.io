

# IO
---
## 格式化
- print("  { } ".format( a )  )
- print("{1},{2},{0}".format(a,b,c))
	- 通过在大括号内填入序号对应format中的元素
	- 要么都有序号，要么都没序号





# Class
---
## 概述
- 类包括属性（变量）和方法，按作用域又可分为类、静态、实例
	- 属性
		- 类属性：定义在类中，函数外部
		- 实例属性：定义在init中
	- 方法
		- 类方法
		- 实例方法
		- 静态方法
- 实例方法调用时自动传入self指示调用方法的实例
- 使用@修饰方法类型
- 定义时使用圆括号表示继承

## 类的创建
### 类公共属性
- 定义在类中的变量，使用类名引用
```python
def cls:
	class_variable='class_variable'
	#在类中，使用cls.class_variable访问此变量
```


### 构造方法与实例属性
- init构造
- 在构造方法中创建的变量是实例属性，使用self引用
```python
def __init__(self,name,age):     #用双下划线表示特殊方法，会自动调用
	self.name=name
	self.age=age
	#使用self.variable_name访问实例属性
```
- 可以在类外部添加实例变量
```python
mc=cls('dh' , 1 )
mc.gender = 'male'
#手动添加实例变量gender，并设定为'male'
```




### 魔法方法call
- 赋予对象被直接调用的能力
- 当一个类实例被当作函数调用时，自动调用call方法
```python
class count:
	def __init__(self):
		self.cnt=0
	def __call__(self):
		self.cnt++
		return self.cnt
```



## 类的通用方法
- isinstance( var , cls )：判断var是否是cls的实例


# zip
---
- 压缩和解压缩



# args和kwargs
- 参数arguments
- 关键字参数key words arguments
```python
# *表示可选参数列表，**表示可选参数字典，args和kwargs是约定的变量名，可以更改
# arg是必须输入的，args和kwargs可以不输入
# kwargs必须在args后面
def func(arg,*args,**kwargs):
	
```


# 断言
---
- assert
	- 判断表达式，若为false，抛出异常