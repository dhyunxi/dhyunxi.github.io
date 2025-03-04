#class

- 张量类


#  定义
---
- pytorch的基础变量类型
- 一个多维数组，支持链式求导

# 成员变量
---
- data：Tensor的值
- dtype：张量的数据类型
- shape：张量的形状
- device：张量所在的设备
- requires_grad：是否开启梯度
- grad：梯度
- grad_fn：创建函数
- is_leaf：是否是叶子

# 类型
---
- torch.FloatTensor：32位浮点
- torch.DoubleTensor：64位浮点
- torch.CharTensor：8位有符号整型
- torch.ByteTensor：8位无符号整型
- torch.BoolTensor：布尔
- 若放在GPU上，则加入cuda，如torch.cuda.FloatTensor

# 创建
---
## 直接创建
- torch.Tensor( data , dtype , device , requires_grad , pin_memory)
	- data可以是list、numpy
- torch.from_numpy( array )
	- 创建的Tensor与array共享内存，改动一个另一个也变化
		- torch.tensor( array ) 没有这种特性

## 依据已有Tensor创建
- torch.ones_like( )：依据形状填充1
- torch.zeros_like( input , dtype , layout)：依据形状填充0
- torch.full_like( fill_value )：依据形状填充给定值
	- fill：填充值

## 特定数值
- torch.zeros( size , * ,out , dtype , layout , device , requires_grad )：填充0
	- out指示存储位置，等价于out=torch.zeros(...)
- torch.ones( size )：填充1
	- kwargs ditto.
- torch.full( size , fill_value)：填充给定值
	- kwargs ditto.
- torch.arange( start , end , step )：依据序列
	- kwargs ditto.
- torch.linspace( start , end , steps=100)：将区间$[start,end)$均分steps份
	- steps是长度而非步长
	- kwargs ditto.
- torch.logspace( start , end , steps , base)：对数均分数列，底为base
	- kwargs ditto.
- torch.eye(n , m)：单位对角矩阵$n\times m$


## 依据概率分布
- torch.normal( mean , std , size , out )：正态分布
	- 若mean和std都是常数，则需指定size
	- 若为list，自动识别
- torch.randn( size )：标准正态分布
- torch.rand( size )：$[0,1)$均匀分布
- torch.randint( lower , upper , size)：整型均匀分布
- torch.randperm( n )：0到n-1的乱序
- torch.bernoulli( p )：伯努利分布，0-1分布


# 操作函数
---
## 填充
- torch.fill
- torch.zero

## 拼接
- torch.cat( tensors , dim , out )
	- 若dim存在，在dim上拼接
	- 若dim不存在，创建新维度
- torch.stack( tensors , dim , out )
	- 在dim上创建新维度，已存在的维度后移

## 切分
- torch.chunk( input , chunks , dim )：平均切分
	- 若不能整除，则最后一个小
- torch.split( tensor , split_size_or_sections , dim )：按split列表切分
	- 若为int，则每份长度为此数
	- 若为list，则按list中的值，若list的元素和不够，则报错

## 索引
- torch.index_select( input , dim , index )：在dim上按index表索引
	- index应为LongTensor类而非list
- torch.masked_select( input , mask , out)：按mask筛选
	- 返回一个1DTensor
- torch.masked_fill()
- torch.masked_scatter( input , mask , target )：掩码替换
- torch.ge( input , other ) ：创建$input\geq other$的mask
	- 类似的有gt、le、lt
	- 重载了运算符，可以直接使用比较运算符创建mask

## 变换
- torch.reshape( input , shape )
	- 新张量与旧张量共享内存
	- 维度填-1代表这个维度自动匹配
		- 不能整除则报错
	- 不支持原地操作
- torch.transpose( input , dim0 , dim1)：交换两个维度
	- torch.t( input )：二维矩阵转置
- torch.permute( input , dims )：维度重排
	- 将维度从0开始编号，dims是编号重排的list
- torch.squeeze( input , dim )：压缩长度为1的轴
	- 不指定dim时，压缩所有长度为1的轴
	- 指定dim时，若此维度长度为1则压缩
- torch.unsqueeze( input , dim)：升维

## 复制
- torch.clone()

## 切换类型和位置
- torch.to

## 数学运算
- torch.add( input , alpha , other )：$input+alpht \times other$
- torch.sub( )
- torch.div( )
- torch.mul( )
- torch.mm( x1 , x2)：哈达玛乘法、逐元素乘法

# 实例方法
---
- **大部分操作函数都定义了相同作用的实例方法**
- .item()
- .detach()：脱离计算图
- .size()
	- shape是属性
- .detach()：从计算图中分离的tensor副本，共享内存
- .detach_()：原位分离
- .random_( generator )：若generator是一个整数，则生成0到generator的随机整数
- .view：相当于reshape


