



# 概念
---
- epoch：完整的训练一次
- batch：一个训练批次，由batch_size个数据项
- iteration：训练一个batch称为1次iteration
- 训练iteration个batch称为一个epoch
- 一个epoch可以有多个iteration



# 核心数据类型
---
- [[torch.Tensor|tenser]]
- torch.half // torch.float16
- torch.float//torch.float32
- torch.double//torch.float64
- torch.uint8
- torch.int8
- torch.short//torch.int16
- torch.int // torch.int32
- torch.long // torch.int64
- torch.bool

# 库
---
- [[torch]]
- [[torch.autograd]]
- [[torch.nn]]



# 计算图
---
- 用来描述运算的有向无环图
- 在反向传播中，非叶子结点的梯度在计算后被释放
- 使用.retain_grad()保留梯度

- 根据图的搭建方式，可分为动态和静态
	- 动态图：运算和搭建同时进行。*灵活、易调节
	- 静态图Tensor Flow：先搭建图，后运算。*高效、不灵活


# 逻辑回归
---
- 二分类模型
## 步骤
- 数据[[Data]]
	- 数据采集
	- 数据划分
	- 数据清理
	- 预处理
- 模型[[Module]]
	- 线性模型
	- 神经网络模型
- 损失函数[[LossFunction]]
	- 均方差
	- 交叉熵
- 优化器[[Optimizer]]
- 迭代训练
	- 前向传播
	- 计算损失
	- 反向传播
	- 更新参数









# 其他
---
## 切片
- `[:,0,1]、[...,0]`
- 使用于ndarray和tensor的切片方法，list不行
```python
#array是2*3*2*2的数组
#考虑为2个项，每项三通道，每通道2*2
array[:,0]  #表示每项的第一个通道
array[:,0,1] #表示每项的第一个通道的第二行
array[...,0] #表示每项，每通道，每行，的第一个元素
#[]中最多可填入4个参数(维数)
#':'代表全部，数字代表对应的项
#'...'代表中间所有
#取出相应的项，组合成新的list
```

## 命名
- 首字母大写的通常是类
- 全小写的通常是包