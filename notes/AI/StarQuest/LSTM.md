
- 长短期记忆神经网络



# 结构
---
## 特点
- 相比RNN，通过两条路径分离长期记忆和短期记忆
- 长期记忆贯穿全局，没有可训练的权重和偏置，因此不会造成梯度爆炸和消失
- 短期记忆有可训练权重和偏置，输入到下一循环
- 使用Sigmoid和Tanh激活函数



## 模型
- 每个iteration有三项输入，分别是长期记忆、短期记忆和直接输入
- 每个iteration有三项输出，分别是长期记忆、短期记忆、和直接输出，非终端结点中直接输出被忽略
- 整个iteration有三个模块
	- 遗忘门
	- 输入门
	- 输出门

### 遗忘门 Forget Gate
$$
\begin{align}


 & output=\mathrm{Sigmoid}(~input*w_{1}+ShortMem_{in}*w_{2}+bias~) \\

 & LongMem=LongMem*output


\end{align}
$$
- input和ShortMem经Linear层和Sigmoid输出output
- output决定了长期记忆存留的百分比

### 输入门 Input Gate
- 记忆门更新长期记忆，由两部分组成
	- 潜在记忆量：使用Tanh激活
	- 潜在遗忘比：使用Sigmoid激活
- 长期记忆增量等于潜在记忆量乘以遗忘比
$$
\begin{align}
 & \text{Potential\_LongMem} =\mathrm{Tanh}\circ \mathrm{Linear}(ShortMem,input) \\
 & \mathrm{Forget\_Percent} = \mathrm{Sigmoid} \circ \mathrm{Linear}(ShortMem,input) \\
& \mathrm{LongMem}=\mathrm{LongMem}+\mathrm{Potential\_LongMem*Forget\_Percent}
\end{align}
$$

### 输出门
- 输出门更新短期记忆，由两部分组成
	- 潜在记忆量：使用Tanh激活
	- 潜在遗忘比：使用Sigmoid激活
- 短期记忆输出等于潜在记忆量乘以遗忘比
- 短期记忆输出是整个iteration的输出

## 长期记忆




- LSTM仿照人脑的记忆，对每个输出设置遗忘单元，增强泛化能力
- LSTM中旧事件的影响系数不是定值，而且不是简单乘法，能有效避免梯度问题
- 短期记忆地位与直接输入相同，昨天发生和今天发生没有太大区别