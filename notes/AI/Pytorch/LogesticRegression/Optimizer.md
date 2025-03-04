
- 优化器



# 概述
---
- 类
- 管理并更新可学习参数的值
## 基本属性
- defaults：超参数
	- 学习率
- state：缓存地址
- param_groups：管理的参数组
	- 整个参数组是以字典为元素的列表
	- 一个group是一个字典，其中有索引'params'
- _ step_count：记录更新次数

## 方法
- zero_grad()：清空梯度
- step()：一步更新
- add_pram_group()：添加参数组
- state_dict()：获取状态字典
- load_state_dict()：加载状态字典

## 实例化
- [parameters]：parameter类构成的list，可以用Tensor替代parameter

## 保存
- torch.save   .pkl文件
- torch.load


# 梯度下降算法
---
## 学习率
- Learning Rate
- 梯度下降中梯度的更新因子
- 控制更新步长

## 动量
- momentum
- 将上次更新信息用于当前更新
$$
\begin{align}
v_{i}=\beta * v_{i-1}+(1-\beta)*\theta_{t} \\
w_{i}=w_{i-1}-LR*v_{i} \\
\end{align}
$$

# 随机梯度下降
---
- optim.SGD
## 参数
- params：管理的参数组
- lr：学习率
- momentum：动量系数
- weight_decay：L2正则化系数
- dampening
- nesterov=False：是否采用NAG


# 学习率调整策略
---
- 在实际学习过程中，后期所需学习率可能要小于前期，可能要求对学习率动态调整

## 基类
- pytorch中采用scheduler类来管理optimizer
- _ LRScheduler是基类
### 属性
- optimizer：关联的优化器
- last_epoch：经历epoch数
- base_lrs：记录初始学习率

### 方法
- step()：更新学习率
- get_lr()：虚函数，计算下一个学习率

### 实现
- 从关联的optimizer的param_groups中提取initial_lr组成list
- 重写step()




- 