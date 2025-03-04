 
- 损失函数


# 概念
---
- 衡量模型输出与真实标签的差异
- 损失函数**Loss Function**：单个样本
	- $Loss=f(\hat{y},y)$
- 代价函数**Cost Function**：全部样本的损失均值
	- $Cost=\frac{1}{N}\Sigma_{i=1}^Nf(\hat{y}_{i},y_{i})$
- 目标函数**Objective Function**：加入正则项，避免过拟合
	- $Obj=Cost+Regularization$


# 常用损失函数
---
- 具体实现参见[[torch.nn.modules.loss|loss模块]]
## L1Loss
- 用L1范数表示距离
$$
\ell(X,Y)=\{l_{1},l_{2},\dots,l_{n} \}^T \quad , \quad l_{i}=|x_{i}-y_{i}|
$$
## SmoothL1Loss
- 在L1范数的基础上，在0附近改用L2范数
- 比均方差更不敏感
- 利于防止梯度爆炸
## 均方差MSE    Mean Square Error
- 用L2范数表示距离
- 
$$
Loss=\Sigma(\hat{y}_{i}-y_{i})^2
$$
## 交叉熵CrossEntropyLoss
### 概念
- 在信息论中，对于实际分布P和预测分布Q，交叉熵为
$$
H(P,Q)=-\sum_{i=1}^n p_{i}*\log q_{i}
$$
- 而实际分布P应当是one-hot分布，即只有一个1，其他均为0，故
$$
H(P,Q)=-\log q_{class} \qquad 其中class代表正确的分类
$$
- $q_{class}$越接近1，交叉熵越接近0，故最小化交叉熵的过程便是优化参数的过程
### 实现
- 交叉熵要求输入是(0,1)的概率，而神经元的输出并无范围，因此CEL函数自带了softmax处理input
- 提供了权值
$$loss(x,class)=weight[class]*\left( -\log \left( \frac{\exp(x[class])}{\Sigma_{j} \exp(x[j])} \right) \right)$$
- 分类任务通常采用交叉熵作为损失函数
	- 这是因为分类的输出往往是(0,1)的概率，若用均方差作为损失函数，loss值会非常小，容易导致梯度消失
	- 同时交叉熵的下凸性利于反向传播


## 二分类交叉熵BCELoss
### 概念
- 在交叉熵中取分布为伯努利分布，则形式退化为
- $$
H(P,Q)=-p\log q-(1-p)\log(1-q)
$$
- 此外，也可以从负对数似然函数来理解
### 实现
- 输入预测概率和实际概率，返回交叉熵
$$
\ell(x, y) = L = \{l_1,\dots,l_N\}^\top, \quad  
    l_n = - w_n \left[ y_n \cdot \log x_n + (1 - y_n) \cdot \log (1 - x_n) \right],  
$$
- 允许提供权值
- 由于处理的是概率，要求输入在$[0,1]$，否则报错

## nn.BCEWithLogitsLoss
- 利用Sigmoid初始化的二分类交叉熵
- 结尾不加sigmoid



## PoissonNLLLoss
- 泊松分布的负对数似然函数

## KLDivLoss
- KL散度
- 需要先归一化
- reduction
	- batchmean：对一个batch取平均
- pytorch的KL散度与信息论不同，需手动log
- 信息论KL
$$
D_{KL}(P||Q)=\mathrm{E}_{X\sim P}\left[ \frac{P(X)}{Q(X)} \right] =\sum_{i}P(X_{i})\log \frac{P(X_{i})}{Q(X_{i})}=\sum_{i} P(X_{i})\log P(X_{i})-D_{CE}
$$
- torch实现
$$
\ell(X,Y)=\{l_{1},l_{2},..,l_{n}\}\quad l_{i}=y_{i}\log y_{i}-x_{i}
$$
- 输入的x要提前log处理
- 可以通过nn.logsoftmax实现

## MarginRankingLoss
- 计算两个向量间的相似度，常用于排序
- 参数
	- margin：边界值
	- reduction
$$
loss(x,y)=\mathrm{max}\{0,-y*(x_{1}-x_{2})+margin \}
$$
- y正：希望x1大于x2，否则有loss（x2-x1）
- y负：希望x1小于x2，否则有loss（x1-x2）
- 对两组数据一一对应计算，得到一个矩阵


## MultiLabelMarginLoss
- 多标签边界损失函数
- 标签形式与CELoss不同，如选1、3类，应是[1,3,-1,-1]
- 