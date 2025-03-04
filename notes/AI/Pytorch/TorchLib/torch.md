#lib

- torch库
>>The torch package contains data structures for multi-dimensional tensors and defines mathematical operations over these tensors. Additionally, it provides many utilities for efficient serialization of Tensors and arbitrary types, and other useful utilities.  
>>
>>It has a CUDA counterpart, that enables you to run your tensor computations on an NVIDIA GPU with compute capability >= 3.0.

# Cagalogue
---
- Tensor及其成员变量和方法，参见[[torch.Tensor]]
- 函数
	- typename
	- is_tensor
	- is_storage
	- get_default_device
	- set_default_device
	- set_default_tensor_type
	- set_default_dtype
- torch.relu
- torch.rrelu
- torch.prelu
- torch.sigmoid



# 函数
---




## torch.manual_seed
- CPU生成随机数的种子，方便重现实验结果

torch.isnan()：是否nan

- torch.max：
	- 取出每行的最大值与indices
	- 会坍缩一个维度
	- keepdim=1以保持维度



torch.round：四舍五入