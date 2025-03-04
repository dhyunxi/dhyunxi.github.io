


- 本节介绍数据模块
- 库torch.utils.data



# 数据模块概述
---
- 数据收集
	- img，label
- 数据划分
	- 训练集、验证集
- 数据读取
	- Dataloader
		- Sampler：生成索引
		- Dataset：根据索引读取图片和标签
- 数据预处理
	- Transforms
	- 中心化、标准化、旋转

## 基础概念
- epoch：所有训练样本都输入
- iteration：一批样本输入
- batchsize：批大小



# 数据读取
---
## Dataloader
- 构建可迭代的数据装载器，每次返回一个batch
- 每一项含两个元素，分别是图片集合和标签集合
	- 图片集合的元素至少含batch_size, C, H, W
- Dataloader( )
	- **dataset**：数据从哪读取，如何读取
	- **batch_size=1**：批大小
	- **shuffle=False**：epoch是否乱序
	- sampler=None：从数据集中抽取样本的策略
	- batch_sampler=None：
	- **num_workers=0**：是否多进程读取
	- **collate_fn**=None：自定义打包方法
		- 主要处理变长序列、复杂结构和特殊预处理
	- pin_memory=False
	- **drop_last=False**：当样本总数不能被整除时，是否舍弃最后一批
	- timeout=0
	- worker_init_fn=None
	- multiprocessing_context=
- 载入流程
	- 生成 batch_size个索引index
		- shuffle为True则打乱
	- 根据索引调用Dataset的getitem方法获取数据和标签
		- sampler、batch_sampler可以自定义取样策略
	- 将获得的batch_size个项打包为一个batch
		- 若给出collate_fn，则按collate_fn打包，否则调用defult_collate，主要应对不规则数据
		- default: 
			- 会进行zip，即若getitem方法返回`data,label`，batch_size=16，则DataLoader返回的迭代对象是2元tuple，第一个元素是第0维度长16的Tensor--data，第二个元素是长16的元组--label
			- zip中，若元素是Tensor，则作stack拼接；若元素不是Tensor，则组合为元组




## dataset
### 概述
- 定义和管理如何获取单个数据及其标签
- 包含加载数据、预处理数据、图像增强等操作
- 返回单个数据样本及标签
```python
class Dataset(object):
	def __getitem__(self,index):
		raise NotImplmentedError

	def __add__(self,other):
		return ConcatDataset([self,other])
```
- dataset是一个抽象类，不能直接使用，需创建子类
- 必须主动定义三个方法
	- __init__
		- 初始化，一般包含获取元素的路径
	- __getitem__
		- 获取元素及其标签的方法
	- __len__
		- 数据的长度及大小
- 读取数据关键：
	- 从哪里读数据
	- 读哪些数据
	- 怎么读数据

![[Pasted image 20250122131155.png]]
### 创建与使用
#### init
- 包含三个参数self、data、label
	- 其中data和label通过getitem方法返回
```python
def __init__(self,Data,Label):
	self.Data = Data
	self.Label = Label
```
#### len
- 返回数据集的大小
```python
def __len__(self):
	return len(self.Data)
```
#### getitem
- 根据输入的索引获取数据内容和标签
- 进行预处理
- 返回内容和标签


### 注意事项
- torch.utils.dataset是一个模块，模块中定义了抽象类Dataset，大小写注意区分
```python
from torch.utils.data import dataset
class MD(dataset.Dataset)   #correct
class MD(dataset)           #false

#可以跳过模块dataset直接引用抽象类Dataset
from torch.utils.data import Dataset
class MD(Dataset)           #correct

#或者不import，但还是需注意大小写
class MD(torch.utils.data.Dataset)
```



# 数据预处理
---
## 数据增强
- 又称数据增广、数据扩增
- 对训练集进行变换，使训练集更丰富，从而使模型更具泛化能力


## transforms库
 - 图像预处理
	- 数据中心化
	- 数据标准化
	- 缩放
	- 裁剪
	- 旋转
	- 翻转
	- 填充
	- 噪声添加
	- 灰度变换
	- 线性变换
	- 仿射变换
	- 亮度、饱和度、对比度变换
- transforms.Compose()
	- 包装一系列指令，依次执行
- transforms.Resize((32,32))
	- 缩放
- transforms.RandomCrop(32,padding=4)
	- 随机裁剪
- transforms.ToTensor()
	- 转化为张量，同时归一化
- transforms.Normalize(norm_mean , norm_std)
	- 标准化


### Normalize
- 逐通道标准化
- 参数
	- mean均值
	- std方差
	- in_place是否原位运算
- $output=(input-mean)/std$
- 流程
	- 合法性判断
	- 判断是否原地操作
	- 获取均值和标准差，转化为张量形式
	- 对Tensor作标准化
- 目的：加速训练收敛速度


### 裁剪crop
- transforms.CenterCrop( size )：中心裁剪
- transforms.RandomCrop：随机裁剪
	- size：大小$size \times size$
	- padding：填充
		- a：上下左右
		- (a,b)：上下b，左右a
		- (a,b,c,d)：左上右下分别abcd
	- pad_if_need：若图像小于size则填充  **size大于图片大小时一定要打开，否则报错
	- padding_mode：
		- const：像素值由fill设定
		- edge像素值由边缘像素决定
		- reflect：镜像，边缘像素不镜像  **边缘作镜子
		- symmetric：镜像，边缘像素镜像   **全对称
	- fill：设置padding的内容
- RandomResizedCrop：随机大小、随机位置裁剪，裁剪完放大到size
	- size：尺寸
	- scale：随机面积比例，默认(0.08,1)
	- ratio：随机长宽比，默认(3/4,4/3)
	- interpolation：插值方法
		- PIL.Image.NEAREST
		- PIL.Image.
		- PIL.Image.BICUBIC
- FiveCrop( size )：上下左右中，五张图片
	- TenCrop：对上述五张图片镜像获得10张图片
		- vertical_flip：垂直翻转？否则水平翻转
	- 返回tuple，不能直接用于transform
	- `transforms.Lambda( lambda crops: torch.stack([transforms.ToTensor()(crop)) for crop in crops]))
	- 正常DataLoader返回是4维的(batch_size,C,H,W)，而FiveCrop返回后是5维(batch_size,ncrops,C,H,W)

### 翻转
-  返回的是img，还需ToTensor
- RandomHorizontalFlip( p )：依概率水平翻转
- RandomVerticalFlip( p )：依概率垂直翻转
 
### 旋转
- RandomRotation：旋转
	- degrees：旋转角度
		- a：在-a到a间选择
		- a，b：在a，b间选择
	- resample=False：重采样方法
	- expand=False：是否扩大图片以保持图片完整出现
		- 只适用于center中心旋转
		- 若选择缩放，后续需resize来保证一个batch的图片大小一致
	- center=None：旋转中心点，默认中心
		- 左上角：(0,0)


### 填充
- transforms.Pad：对图片边缘填充
	- padding
	- fill=0：(R,G,B) or (Gray)
	- padding_mode
		- constant
		- edge
		- reflect：反射，复制边缘
		- symmetric：对称，不复制边缘

### 亮度
- transforms.ColorJitter：亮度、对比度、饱和度、色相
	- brightness：亮度
		- a：从$max\{0,1-a\},1+a$中随机选取
		-  (a,b)：从a和b间选取
	- contrast：对比度
	- saturation：饱和度
	- hue：色相
		- a：-a到a间
		- a，b：a到b间
		- 注：绝对值不超过0.5

### 灰度
- RandomGrayscale：转化为灰度图
	- num_output_channles：输出通道数，1或3
	- p：概率
- Grayscale：概率为1的RandomGrayscale

### 仿射变换
- 旋转、平移、缩放、错切、翻转
- RandomAffine
	- degrees：旋转角度
	- translate：平移区间
		- (w,h)：宽和高
	- scale：缩放比例，0到1
	- shear：错切角度
		- a：仅在x轴错切，-a到a
		- (a,b)：a设置x，b设置y
		- (a,b,c,d)：a、b设置x，c、d设置y
	- resample
	- fill_color：填充颜色

### 遮挡
- 作用于张量，放在ToTensor之后
- RandomErasing：随机遮挡
	- p：概率
	- scale：遮挡面积
	- ratio：长宽比
	- value：遮挡像素
		- 可以设置为'random'
### Lambda
- transform.Lambda(lambda *args* :  ... )

### 逆操作
- transform_invert

### Transforms方法选择
- transforms.RandomChoice([ ],p)：从一系列方法中随机挑选一个Transform
- transforms.RandomApply([ ],p)：依据概率执行一组操作Transform
- transforms.RandomOrder：打乱顺序

## 自定义Transforms
- 接受一个参数，返回一个参数
- 不能更改数据类型
- 通过类实现多参数传入
```python
#椒盐噪声：随机出现白点和黑点
#Signal-Noise Rate，SNR：信噪比
class YourTransForm(object)
	def __init__(self,snr,p=0.9)
		assert isinstance(snr,float) or (isinstance(p,float))
		self.snr=snr
		self.p=p
		
	def __call__(self,img):
		if random.unform(0,1) < self.p:
			img_=np.array(img).copy()
			h,w,c=img_.shape
			signal_pct=self.snr
			noise_pct=1-self.snr
			mask = np.random.choice((0,1,2),size=(h,w,1),p=[signal_pct,noise_pct/2.,noise_pct/2.])
			mask = mp.repeat(make,c,axis=2)
			img_[mask==1]=255
			img_[mask==2]=0
			return Image.fromarray(img_.astype('uint8')).convert('RGB')
		else:
			return img
```


## 数据增强策略
- 让训练集与测试集更接近
	- 空间位置
	- 色彩
	- 形状
	- 上下文



# 实例化
---
- MyDataset( data_dir , transform )
- DataLoader( dataset  ,  )