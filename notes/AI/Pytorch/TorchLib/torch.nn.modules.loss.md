
#module 

- 损失函数


# Catalogue
- 共22项
- 基类
	- Loss
	- WeightedLoss
- 常用函数
	- L1
		- L1Loss
		- SmoothL1Loss
	- NLL
		- NLLLoss
		- NLLLoss2d
	- L2
		- MSELoss：均方差
	- 交叉熵
		- CrossEntropyLoss
		- BCELoss：二分类交叉熵
		- BCEWithLogitsLoss
	- 散度
		- KLDivLoss
	- PoisonNLLLoss
	- GaussianNLLLoss
	- Margin
		- MarginRankingLoss
		- MultiLabelMarginLoss
		- SoftMarginLoss
		- MultiLabelSoftMarginLoss
		- MultiMarginLoss
		- TripletMarginLoss
	- Embedding
		- HingeEmbeddingLoss
		- CosineEmbeddingLoss
	- CTCLoss




# 基类
---
## Loss
- Module子类
- 损失函数基类
- 定义了reduction

## WeightedLoss
- Loss子类
- 带权损失函数基类

# 常用函数
---
## L1Loss
- Loss子类
### 实例参数
- reduction
### 调用参数
- input
- target
### 实现
$$
\ell(X,Y)=\{l_{1},l_{2},\dots,l_{n} \}^T \quad , \quad l_{i}=|x_{i}-y_{i}|
$$

## SmoothL1Loss
- Loss子类
- 将L2范数和L1范数结合以平滑0附近的函数
- 敏感性低于均方差
### 实例参数
- reduction='mean'
	- 'none'
	- 'mean'
	- 'sum'
- beta
	- 必须使非负的
### 调用参数
- input
- target
### 实现
$$
\ell(x,y)=L=\{l_{1},l_{2},\dots,l_{n}\},~~~\text{if reduction=`none'}
$$
$$
l_{i}=\begin{cases}
\frac{1}{2\beta}(x_{i}-y_{i})^2, & \text{if }|x_{i}-y_{i}|<\beta \\
|x_{i}-y_{i}|-\frac{1}{2}\beta, & \text{otherwise}
\end{cases}
$$
- 在0附近平滑
$$
\ell(x, y) =  
\begin{cases}  
    \operatorname{mean}(L), &  \text{if reduction} = \text{`mean';}\\    \operatorname{sum}(L),  &  \text{if reduction} = \text{`sum'.}\end{cases}
$$
## 均方差MSELoss
- Loss子类
### 实例参数
- reduction='mean'
	- none
	- sum
	- mean
### 调用参数
- input：预测值
- target：真实值
### 实现
$$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad
        l_n = \left( x_n - y_n \right)^2
$$
$$
\ell(x, y) =  
\begin{cases}   
    L, &        \text{if reduction = `none';}                             \\
    \operatorname{mean}(L), &  \text{if reduction} = \text{`mean';}\\    \operatorname{sum}(L),  &  \text{if reduction} = \text{`sum'.}\end{cases}
$$

### 实例
```python
input:Tensor
label:Tensor
net_loss=nn.MSELoss()  # 实例化
output=net_loss(input,label)
```



## nn.NLLLoss
- Loss子类
- Negtive Log Likelihood Loss
- CrossEntropyLoss中的取负号操作
### 实例参数
- weight：1DTensor
- ignore_index
- reduction
	- none
	- sum
	- mean
### 调用参数
- input：
	- 期待是log-probabilities，其实没关系
	- 至少1个维度
		- 若只有1个维度，视为class
		- 若至少2个维度，认为第一个维度是batch，第二个维度是class
- target
	- 长度为batch_size的LongTensor
	- 若input只有一个维度，则为标量，**不能有维度**
	- 每一项是小于class_size的非负整数，代表了对input的第i个batch中的哪一项取负号
- output：'none'
	- 在input基础上坍缩class维度
### 实现
  $$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad    l_n = - w_{y_n} x_{n,y_n}, \quad    w_{c} = \text{weight}[c] \cdot \mathbb{1}\{c \not= \text{ignore\_index}\},  
$$
$$
\ell(x, y) = \begin{cases}        \sum_{n=1}^N \frac{1}{\sum_{n=1}^N w_{y_n}} l_n, &        \text{if reduction} = \text{`mean';}\\        \sum_{n=1}^N l_n,  &        \text{if reduction} = \text{`sum'.}    \end{cases}  
$$
- 将对应类别的x取负号
### 实例
```python
#实例化loss
loss_fn = nn.NLLLoss(reduction='none')
#输入
input = torch.randn(3,5)
log_input = nn.LogSoftmax(dim=1)(input)
#target
target=torch.randint(0,6,3)
#输出
output = loss_fn(log_input,target)
```

## NLLLoss2D
- NLLLoss 子类





## 交叉熵CrossEntropyLoss
- WeightedLoss子类
### 实例参数
- weight=None：各类别Loss权值
	- 1DTensor：长度等于Class数目
- ignore_index=None：忽略某个类别
- reduction='mean：计算模式
	- 'none'：逐个计算，返回Tensor
	- 'sum'：求和，返回标量
	- 'mean'：加权平均
- label_smoothing
### 调用参数
- input
	- 无需标准化，CELoss自带Softmax
	- 至少一个维度
		- 若只有一个维度，维度长即为class数目
		- 若有不少于两个维度，则认为第一个维度是batch，第二个维度是class
- target
	- 有两种形式
		- class indices：类别序号
			- 长度为batch_size的LongTensor
			- 若input只有一个维度，则为标量，**不能有维度**
			- 每一项是小于class_size的非负整数，代表了取input的第i个batch中的哪一项
		- class probabilities：类别概率
			- 与input的shape相同，且每一项在$[0,1]$上
	- 使用类别序号在大多数时候要优于类别概率，除非每个batch的每个class都需要label，例如blended label、label smoothing
### 实现
- target=class indices
$$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad  
l_n = - w_{y_n} \log \frac{\exp(x_{n,y_n})}{\sum_{c=1}^C \exp(x_{n,c})}  
\cdot \mathbb{1}\{y_n \not= \text{ignore\_index}\}
$$
$$
\ell(x, y) = \begin{cases}  
    \sum_{n=1}^N \frac{1}{\sum_{n=1}^N w_{y_n} \cdot \mathbb{1}\{y_n \not= \text{ignore\_index}\}} l_n, &     \text{if reduction} = \text{`mean';}\\      \sum_{n=1}^N l_n,  &      \text{if reduction} = \text{`sum'.}  \end{cases}
$$
	- 等价于LogSoftmax+NLLLoss
- target=class probabilities
$$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad  
l_n = - \sum_{c=1}^C w_c \log \frac{\exp(x_{n,c})}{\sum_{i=1}^C \exp(x_{n,i})} y_{n,c}
$$
$$
\ell(x, y) = \begin{cases}  
    \frac{\sum_{n=1}^N l_n}{N}, &     \text{if reduction} = \text{`mean';}\\      \sum_{n=1}^N l_n,  &      \text{if reduction} = \text{`sum'.}  \end{cases}
$$
- output在input基础上坍缩class维度
### 实例
- 实例化
- 输入input和label
	- label必须是torch.long
- weight是一个Tensor，项数必须与类数一致
- 类序号必须从0开始
```python
input = torch.tensor([[ 0.8082,  1.3686, -0.6107],#1
                      [ 1.2787,  0.1579,  0.6178],#0
                      [-0.6033, -1.1306,  0.0672],#2
                      [-0.7814,  0.1185, -0.2945]])#1
target = torch.tensor([1,0,2,1])#假设标签值(不是one-hot编码，直接代表类别）
loss = nn.CrossEntropyLoss()#定义交叉熵损失函数（自带softmax输出函数)
output = loss(input, target)
print(output)#tensor(0.6172)

```


## 二分类交叉熵 BCELoss
- WeightedLoss子类
### 实例参数
- weight
- reduction='none'
	- 'none'
	- 'mean'
	- 'sum'
### 调用参数
- input
	- 要求(0,1)
- target
	- 要求(0,1)
### 实现
$$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad  
    l_n = - w_n \left[ y_n \cdot \log x_n + (1 - y_n) \cdot \log (1 - x_n) \right],  
$$
$$
 \ell(x, y) = \begin{cases}        \operatorname{mean}(L), & \text{if reduction} = \text{`mean';}\\        \operatorname{sum}(L),  & \text{if reduction} = \text{`sum'.}    \end{cases}
$$
- 为了防止$x_{n}=0$导致$\log x_{n}=-\infty$，自动约束log的输出大于等于-100
### 实例
```python
m = nn.Sigmoid()  
loss = nn.BCELoss()  
input = torch.randn(3, 2, requires_grad=True)  
target = torch.rand(3, 2, requires_grad=False)  
output = loss(m(input), target)  
output.backward()
```


## nn.BCEWithLogitsLoss
- Loss子类
- 在入口处添加了Sigmoid的BCELoss



## PoissonNLLLoss
- Loss子类
### 实例参数
- log_input=True：输入是否为对数形式
- full=False：是否计算完整的loss，即是否纳入$\log(target!)$
- eps=1e-08：修正项，防止log0=nan
- reduction
	- 'none'
	- 'mean'
	- 'sum'

### 调用参数
- input
- target：
	- shape与input相同
- output
	- shape与input相同


### 实现
- 泊松分布的负对数似然函数
$$
loss(input,target )=\begin{cases}
\exp(input)-target*input,  &  \text{log\_input=True} \\
input-target*\log(input+eps ), & \text{log\_input=False}
\end{cases}
$$
- 若full=True，则加入$\log(target!)$项
### 实例
```python
loss = nn. PoissonNLLLoss()
log_input = torch. randn(5, 2, requires_grad=True)
target = torch. randn(5, 2)
output = loss(log_input, target)
output. backward()
```

## KLDivLoss
- Loss子类
### 实例参数
- reduction
	- ‘none'
	- 'mean'
	- 'sum'
	- =='batchmean'：对批求平均==
- log_target
	- 为了避免数值下溢
### 调用参数
- input
	- 为了避免数值下溢，input应当是log-space
- target
	- 根据log_target判断是否为log-space
- output
### 实现
- KL散度
- 与信息论中的KL散度不同，input已是log形式，target依据log_target判断是否为log形式
$$
\text{log\_target=False:}~~~~~~~\ell(X,Y)=\{l_{1},l_{2},..,l_{n}\},\quad l_{i}=y_{i}*(\log y_{i}-x_{i})
$$
### 实例
```python
>>> kl_loss = nn. KLDivLoss(reduction="batchmean")
>>> # input should be a distribution in the log space
>>> input = F. log_softmax(torch. randn(3, 5, requires_grad=True), dim=1)
>>> # Sample a batch of distributions. Usually this would come from the dataset
>>> target = F. softmax(torch. rand(3, 5), dim=1)
>>> output = kl_loss(input, target)
>>> kl_loss = nn. KLDivLoss(reduction="batchmean", log_target=True)
>>> log_target = F. log_softmax(torch. rand(3, 5), dim=1)
>>> output = kl_loss(input, log_target
```
