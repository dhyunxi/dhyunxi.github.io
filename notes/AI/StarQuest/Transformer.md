
- 本文记录StarQuest学习笔记
- [【中英双语】ChatGPT背后的数学原理是什么？带你看懂Transformer模型的数学矩阵实现！_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1UDyUYuEfQ?spm_id_from=333.788.videopod.sections&vd_source=d5dfd0b2a455103b5d899d64b40b2576)



# 概述
---
- Transformer是为自然语言处理设计的模型，但计算机无法直接处理文字和语句
- Transformer的输入核心在于embedding机制，即将token转化为向量，从而将文字转化为数字，提供给计算机处理
- 为了学习到文义特质，embedding应有如下性质
	- 文义相近的词较邻近
	- 一个词可以有多个文义
- Transformer的学习核心在于自注意力机制




# Embedding
---
- 词嵌入机制可分为两个部分
	- 分词
	- word2vec


## word2vec
- 初始化词向量
- 损失函数：Cross Entropy Loss
- 反向传播

- Continuous Bag of Words 连续词袋
	- FIB预测，利用两侧的词预测中间
- Skip Gram 跳跃模型
	- 使用中间的词预测周围
- 负采样：对于某个输入，随机选择不想预测的单词计算损失







# 自注意力机制
---
- 通过自编码机将每个token转化为一个n维向量，即词嵌入
- 以Attention注意力值衡量编码的优劣
## Encoder
- 将token通过线性层变换为Key矩阵和Query矩阵
- Query乘以Key的转置 ，并对Key序作Softmax标准化
- 乘以V得到Attention


## Decoder



### Teacher Forcing机制
- 对立于Free Running
- 在循环连接网络(即输出会返回给模型)中，用教师信号替换上一层的输出信号，来避免某一层垃圾信号的影响