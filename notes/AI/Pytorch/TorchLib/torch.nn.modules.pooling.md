

# 概述
- 池化运算：对信号进行收集并总结，类似水池收集水资源
	- 最大值池化
	- 平均值池化
- 用于冗余信息剔除，减少计算量



# 池化-下采样
---
## nn.MaxPool2d
- 对二维信号最大值池化
- 参数
	- kernel_size
	- stride=None：**步长默认等于kernel_size而非1**
	- padding=0
	- dilation=1
	- ceil_mode=False：向上取整
	- return_indices=False：记录池化像素索引
		- 若设置为True，则返回两个Tensor，第一个是池化后的input，第二个是记录取值位置的indices

## nn.AvgPool2d
- 对二维信号平均池化
- 参数
	- kernel_size
	- stride=None
	- padding=0
	- ceil_mode=False
	- count_include_pad=True
	- divisor_override=None：除法因子



# 反池化-上采样
---
## nn.MaxUnpool2d
- 对二维信号反最大池化
- 参数
	- kernel_size
	- stride
	- padding
- 利用最大值池化中的索引标记return_indices=True可以记录最大值的位置
	- 在call时可传入indices