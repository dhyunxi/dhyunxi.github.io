
#pack


- 神经网络的工具包
- from torch.nn.modules import * 
	- modules被全部导入，索引时可以省略modules


# Catalogue
---
- parameter
	- Buffer
	- Parameter
	- UninitializedBuffer
	- UninitializedParameter
- modules：网络层模型模块
- attention
- functional：函数模块
- init：参数初始化模块
- parallel
	- DataParallel
- utils：工具模块

## Highlights
- nn.Parameter：张量子类，表示可学习参数，如weight、bias
- nn.Module：网络层基类，管理网络属性
- nn.functional：函数实现模块，如卷积、池化、激励函数
- nn.init：参数初始化模块





# Others
---
- model.train()
	- 训练模式
	- 启用dropout和normalization
- model.eval()
	- 评估模式


## Softmax
### 若指定的dim不是最小维度
```python
input=torch.Tensor([[1,2,3],[4,5,6]])  #2DTensor
output=nn.Softmax(dim=0)(input)   #对非底层维度做softmax
```
- 计算逻辑是将一行作为一个整体，计算exp后的和r1和r2
- output中第一行的值相同，等于$\frac{r_{1}}{r_{1}+r_{2}}$，第二行为$\frac{r_{2}}{r_{1}+r_{2}}$
- shape与input一致