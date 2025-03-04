
#pytorch  #卷积



# 卷积
---
## 概念
- 卷积运算：卷积核在输入信号上滑动，在相应位置上乘加
- 卷积核：又称滤波器、过滤器，可认为是某种模式、某种特征
- 卷积过程类似与用一个模板去图像上找相似的区域，与卷积核越相似，激活值越高，从而实现特征提取
- 卷积维度：卷积核滑动的维度

## nn.Conv2d
- 对多个二维信号进行二维卷积
- 参数
	- in_channels：输入通道数
	- out_channels：输出通道数
	- kernel_size：卷积核尺寸
	- stride：步长
	- padding：填充
	- dilation：空洞卷积大小
	- groups：分组卷积
	- bias：偏置
- 尺寸计算
$$
out_{size}=\frac{{In_{size}-kernel_{size}}}{stride}+1
$$
$$
完整版：
H_{out}=\left\lfloor  \frac{{H_{in}+2\times padding-dilation\times(kernel_{size}-1)-1}}{stride}+1  \right\rfloor 
$$
- 2维卷积中，卷积核是一个4维张量
	- 第一个维度是卷积核数量
	- 第二个维度是输入通道数
	- 后两个维度是卷积核shape
- 输入应当是3D或4D张量
	- 3D：第一维是BatchSize，后两维是信号
	- 4D：第一维是BatchSize，第二维是in_channel，后两维是信号



# 转置卷积
---
## 概念
- 正常卷积将尺寸映射的更小，而转置卷积将图像映射到更大的尺寸
- 又称部分跨越卷积
- 常用于对图像进行上采样

## 例
- 计算机进行卷积时往往将图片和卷积核展开为矩阵
- 正常卷积：
	- 若有$4\times 4$图片，卷积核$3\times 3$，padding=0，stride=1
		$图像：I_{16*1} \quad 卷积核：K_{4*16}  \quad 输出：O_{4*1}=K_{4*16}*I_{16*1}$
- 转置卷积：
	- 若有$2\times 2$图片，卷积核$3\times 3$，padding=0，stride=1
		$图像：I_{4*1} \quad 卷积核：K_{16*4}  \quad 输出：O_{16*1}=K_{16*4}*I_{4*1}$

## nn.ConvTranspose2d
- 转置卷积
- 参数
	- 与Conv2d一致
- 尺寸计算
	- 与Conv2d相逆


## 棋盘效应