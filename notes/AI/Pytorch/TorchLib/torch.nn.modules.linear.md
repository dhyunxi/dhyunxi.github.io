
- 线性层模块

# Catalogue
---
- Indentity
- Linear
- BiLinear
- LazyLinear


# 概述
---
- 线性层，又称全连接层，其每个神经元与上一层所有神经元连接
- 分类：以下都是库linear中的类(Module子类)
	- Linear
	- Bilinear
	- LazyLinear
	- Indentity


# 结构
---
- Input是一个行向量
- 线性变换利用矩阵表示
- Hidden隐藏层
- Output输出
$$
Output_{1\times b}=Input_{1 \times a}\times M_{a \times b} + Bias_{1\times b}
$$



# nn.Linear
---
- 参数
	- in_features：输入节点数
	- out_features：输出节点数
	- bias=True：是否采用偏置
- 计算
$$
y = x\times W^T+bias
$$
## 实现
```python
inputs = torch.tensor([[1,2,3]]) # 第一个维度是batch_size
linear_layer=nn.Linear(3,4)      #线性层构建
linear_layer.weight.data = torch.tensor([[1,1,1],
										 [2,2,2],
										 [3,3,3],
										 [4,4,4]],dtype=float)
linear_layer.bias.data.fill_(1.)
output=linear_layer(inputs)
```
