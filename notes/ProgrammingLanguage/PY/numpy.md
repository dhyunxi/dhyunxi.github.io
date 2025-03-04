

# ndarray
---

## 参数
- dtype
- order：'C'或‘F'


## 成员
- shape
	- 可以用来调整形状 a.shape=
- ndim
- itemsize
- flag


## 方法
### 创建
#### 空
- np.empty
#### 数值
- np.ones
- np.ones_like
- np.zeros
- np.zeros_like
- np.eye
- np.full
#### 随机
- np.random.rand：0到1均匀分布
- np.random.uniform：均匀分布
- np.random.randint：均匀分布整数
- np.random.randn：标准正态
- np.random.normal：正态
- np.random.shuffle：打乱
- np.random.seed：设置种子
- random_sample：随机浮点


#### 变换
- reshape：改变维数，COPY
- resize：改变维数，修改自己
- T：转置
- ravel：展平，不COPY
- flatten：展平，COPY
- squeeze：压缩
- transpose：交换轴