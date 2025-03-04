
- 本节介绍模型模块




# 概述
---
- 模型对应了神经网络中的网络层，例如全连接层、卷积层等等
- 网络层处理由两部分组成，分别是前向传播方法和权值初始化
## 步骤
- 模块创建
	- 构建网络层
	- 拼接网络层
- 权值初始化
	- Xavier、Kaiming、均匀分布、正态分布


# nn模块
---
- `from torch import nn`
- torch.nn是构建神经网络的工具包，其中较为重要的有
	- nn.Parameter：张量子类，用来装载可学习参数，如weight、bias
	- nn.Module：网络层基类，封装了管理参数的方法，需重写init和forward
	- nn.functional：函数实现模块，包含常用函数，如ReLU、Softmax等
	- nn.init：参数初始化模块
- nn.moduls是网络层工具包，其中含有各类网络层基类及其管理工具
	- 网络层基类module
	- 线性变换linear
	- 激活函数activation
	- 容器container
	- 卷积网络conv
	- 正则化dropout
	- 损失函数loss
	- 标准化normalization
	- 填充padding
	- 池化pooling
	- 变形器transformer
- 与Dataset一样，需要注意大小写。modules是模块，Module是其中的类

## nn.Module
- 网络层基类，用于继承构建网络层，定义了call函数实现前向传播
- 属性：每个属性是一个字典，用来管理各类参数
	- parameters：存储管理nn.Parameter，即模型的可学习参数
	- modules：存储管理nn.Module类
	- buffers：存储管理缓冲属性
	- *** _ hooks：存储管理钩子函数
- 嵌套：
	- Module子类可以通过modules参数实现树形嵌套，即小网络可组合成大网络
	- 这也可以由Sequential实现
- 拦截：
	- Module类会拦截所有类属性赋值操作
	- 判断待赋值的类型
		- parameter：加入parameters字典
		- Module：加入modules字典
	- 加入对应的属性字典以管理
	- 以`self.conv = nn.Conv2d(1,6,5)`为例
		- 实例化nn.Conv2d(1,6,5)
		- 拦截赋值
			- 判断nn.Conv2d(1,6,5)是Module类
			- 将conv加入字典modules
		- 完成赋值
- 必须实现init和forward函数
### 实例方法
- self.modules()
	- 返回一个迭代器，层序遍历所有modules参数
	- ==注意是层序所有，无需多重循环==




# 模型创建
---

### 构建子类LeNet
- init初始化：构建模块
- forward前向传播：拼接模块
- LeNet
	- 卷积层+池化层+卷积层+池化层+全连接层+全连接层+全连接层
```python
import torch.nn as nn
import torch.nn.functional as F
class LeNet(nn.Module):   #继承nn.Module
	#网络层创建
	def __init__(self,classes):
		super(LeNet,self).__init__()     #继承init
		self.conv1 = nn.Conv2d(3,6,5)    #卷积层1
		self.conv2 = nn.Conv2d(6,16,5)   #卷积层2
		self.fc1 = nn.Linear(16*5*5,120) #全连接层1
		self.fc2 = nn.Linear(120,84)     #全连接层2
		self.fc3 = nn.Linear(84,classes) #全连接层3
	#网络层拼接：前向传播
	def forward(self,x):
		out = F.relu(self.conv1(x))
		out = F.max_pool2d(out,2)
		out = F.relu(self.conv2(out))
		out = F.max_pool2d(out,2)
		out = out.view(out.size(0),-1)
		out = F.relu(self.fc1(out))
		out = F.relu(self.fc2(out))
		out = self.fc3(out)
		return out
```
- forward通常用call方法包装

## 神经网络层类型
- 全连接层
- 卷积层
- 池化层





## Containers
### 概述
- nn.Module的容器，用于包装网络层
- nn.Sequetial：按顺序包装
	- 可以输入OrderedDict类型
	- 顺序性
	- 自带forward
- nn.ModuleList：像list一样包装，以迭代方式调用
	- append()：在尾部添加元素
	- extend()：拼接两个ModuleList
	- insert()：在指定位置插入网络层
- nn.ModuleDict：像字典一样包装
	- clear()
	- items()
	- keys()
	- values()
	- pops()

### Sequential例：利用包装LeNet
- 构建block
- 在卷积神经网络中，可以把卷积池化层看作特征提取器，把全连接层看作分类器，分开包装
```python
class LeNet_Sequential:
	def __init__(self,classes):
		super(LeNet_Sequential,self).__init__()
		self.features = nn.Sequential(  # 包装特征提取器
			nn.Conv2d(3,6,5),    #卷积层1
			nn.ReLU(),
			nn.MaxPool2d(kernal_size=2,step=2),
			nn.Conv2d(6,16,5)   #卷积层2
			nn.ReLU(),
			nn.MaxPool2d(kernal_size=2,step=2)
		)
		
		self.classifier = nn.Sequential(
			nn. Linear(16*5*5,120)#全连接层1
			nn.ReLU()
			nn.Linear(120,84)     #全连接层2
			nn.ReLU()
			nn.Linear(84,classes) #全连接层3
		)
	def forward(self,x):
		x=self.features(x)
		x=x.view(x.size(0),-1)
		x=self.classifier(x)
		return x
```
- 不再需要使用nn.modules.functional，只需将层与运算排序，Sequential会自动运行
- 大大简化了forward，因为Sequential自动实现层间forward

### ModuleList
- 通过列表表达式构建网络层，用于大量重复网络的构建
```python
def ModuleList(nn.Module):
	def __init__(self):
		super(ModuleList,self).__init__()
		self.linears = nn.ModuleList([nn.Linear(10,10) for i in range(20)])
	def forward(self,x):
		for i,linear in enumerate(self.linears):
			x=linear(x)
		return x
```


### ModuleDict
- 以索引方式调用网络层，用于可选择的网络层
- 使forward可动态变化
```python
def ModuleDict(nn.Module)
	def __init__(self):
		super(ModuleDict,self).__init__()
		self.choices = nn.ModuleDict([
			'conv':nn.Conv2d(10,10,3)
			'pool':nn.MaxPool2d(3)
		])
		self.activations = nn.ModuleDict([
			'relu':nn.ReLU()
			'orelu':nn.PReLU()
		])
	def forward(self,x,choice,act):
		x=self.choices[choice](x)
		x=self.activations[act](x)
		return x
```

## AlexNet
---
- 采用ReLU：替换饱和激活函数，减轻梯度消失
- 采用LRN：归一化，减轻梯度消失
- Dropout：正则化，提高鲁棒性
- Data Augmentation：TenCrop，色彩修改




# 权值初始化
---
## 梯度消失和爆炸
- 在神经网络中，若某层的权值过分大，则求梯度时这部分会使偏导过分大，导致梯度爆炸；同样的，过小的权值会使梯度消失



## 分布一致性
### 均值一致性
- 标准化输入使均值为0
- 线性变换不改变均值
- 采用均值为0的激活函数，如tanh

### 方差爆炸
- 假设每层n个神经元，标准差每传播一层就会扩大$\sqrt{ n }$倍
	- $\mathrm{D(X+Y)=D(X)+D(Y)}$
	- $\mathrm{D(X*Y)=D(X)*D(Y) \quad st.}均值为0$
	- $\mathrm{D(Output)}=\Sigma_{i=1}^n \space\mathrm{D(Input)*D(W_{:,i})}$
	- 若$\mathrm{D(W_{:,i})=D(Input)=1}$，则$\mathrm{D(Output)}=n$
- 为了让方差稳定在1可令$\mathrm{D(W_{:,i})=\frac{1}{n}}$
- 但若有激活函数，如tanh等，会使方差过小趋于0

### Xavier初始化
#### 推导
- 针对饱和激活函数
- 方差一致性：保持数据尺度维持在恰当范围，通常为1  
- 希望有
$$
\begin{align}
n_{i}*D(W)=1 \\
n_{i+1}*D(W)=1 \\
\Rightarrow D(W)\approx \frac{2}{n_{i}+n_{i+1}}
\end{align}
$$
- 若W采取均匀分布$\mathrm{W \sim U[-a,a]}$，则$\mathrm{D(W)}=\frac{(-a-a)^2}{12}=\frac{a^{2}}{3}$
- 令上两式相等，求得
$$
a=\frac{\sqrt{ 6 }}{\sqrt{ n_{i}+n_{i+1} }}
$$
#### 实现
```python
nn.init.xavier_uniform(m.weight.data.gain=tanh_gain)
#gain是激活函数的贡献
```

#### 缺陷
- 只适用与饱和激活函数，不适用于ReLU

### Kaiming初始化
- 方差一致性
- 激活函数：ReLU及其变种

#### 推导
$$
\begin{align}
\mathrm{D(W)=\frac{2}{n_{i}}} \\
\mathrm{D(W)=\frac{2}{(1+a^2)*n_{i}}}
\end{align}
$$


## 其他初始化方法
### Xavier
- Xavier均匀分布
- Xavier标准正态
### Kaiming
- Kaiming均匀
- Kaiming标准正态
### 概率
- 均匀
- 正态
- 常数
### 矩阵
- 正交矩阵
- 单位矩阵
- 稀疏矩阵

## nn.init.calculate_gain
- 计算方差变化尺度
- 参数
	- nonlinearity：激活函数名称
	- param：激活函数参数