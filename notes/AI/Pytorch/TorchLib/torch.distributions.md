
#pack 

- distributions包含有各类概率分布以及取样函数

# Module Catalog
---
- .distribution：只含有一个基类Distribution







# Distribution基类
---
## init args
- batch_shape: torch.Size
- event_shape
- validate_args=None

## Property
- arg_constraints: Dict[Str, Constraint]
	- 参数服从的约束
	- 字典：{ 键：参数，值：参数服从的约束 }
- batch_shape：torch.Size


## Function
- cdf( value )
	- 返回value处的概率质量或累积概率密度
	- args: Tensor
	- 
