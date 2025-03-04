
#module

- 网络层参数的初始化方法
- 由于是初始化，所有方法都采用no_grad

# Catalogue
---
- calculate_gain
- 基于数值或概率
	- uniform_：等概率初始化
	- normal_：正态
	- constant_：常值
- Xavier
	- xavier_uniform_
	- xavier_normal_
- Kaiming
	- kaiming_uniform_
	- kaiming_normal_
- 矩阵
	- orthoganal_
	- sparse_
	- eye_
	- dirac_
	- 





# calculate_gain
---
## 作用
- 获取某个函数对方差的增益值
	- Linear : 1
	- Indentity : 1
	- Conv{1,2,3}D : 1
	- Sigmoid : 1
	- Tanh : 5/3
	- ReLU : $\sqrt{ 2 }$
	- Leaky ReLU : $\sqrt{ \frac{2}{1+negative\_slope^2}}$
## 参数
- nonlinearity：函数名，只能在上述函数中选择
- param=None：函数参数

## 使用
```python
tanh_gain=nn.init.calculate_gain('tanh') #获取tanh的贡献
a*=tanh_gain #乘入
```


# 基于数值和概率的初始化
---
## uniform_
- 等概率初始化
```python
def uniform_(  
    tensor: Tensor,     # 待初始化参数
    a: float = 0.0,     # 范围下界
    b: float = 1.0,     # 范围上界
    generator: _Optional[torch.Generator] = None,    # 生成器
) -> Tensor:
```