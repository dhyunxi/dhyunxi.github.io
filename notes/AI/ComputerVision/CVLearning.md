



# 读写 IO
---
## 窗口创建与销毁
- cv2.namedWindow：创建一个自命名窗口
	- win_name：窗口名称
	- flags：窗口类型
		- cv2.WINDOW_AUTOSIZE：自适应窗口，无法resize
		- cv2.WINDOW_NORMAL：标准窗口，允许resize
- cv2.imshow：在指定窗口中展示图片
	- win_name：窗口名称
	- mat：要显示的图片矩阵，ndarray: dtype=uint8
- cv2.resizeWindow：改变窗口大小
	- win_name：窗口名称
	- size：width , hight
- cv2.destroyWindow：销毁一个窗口
	- win_name：销毁的窗口名称
- cv2.destroyAllWindows：销毁所有窗口
- cv2.waitKey：等待按键或一定时间
	- 0: 等待一个按键
	- 正整数：等待以毫秒为单位的时间

## 图片读入与显示
- cv2.imread
	- path：图片路径
	- flags
		- cv2.IMREAD_GRAYSCALE：以灰度图形式读入
	- 读入的图片以ndarray: uint8形式储存
	- 通道顺序BGR
- 按下q键使窗口销毁
```python
def cv_show(name,img):
    cv2.imshow(name,img)
    key=cv2.waitKey(0)
    if key&0XFF == ord('q'):
        print('exit window')
        cv2.destroyAllWindows()
```
 

## 图片保存
- imwrite
	- file_name：保存名称
	- target：待保存图片

## 视频和摄像头采集
- [class] cv2.videoCapture
	- path：视频路径或摄像头ID
	- 返回一个videoCapure对象
- self.read()
	- 返回一个标记和一帧，标记指示是否成功读取
- self.isOpened()
	- 是否打开成功
```python

cv2.namedWindow('video',cv2.WINDOW_NORMAL)
cv2.resizeWindow('video',640,480)
cap = cv2.videoCapture(0)  # 提取ID为0的摄像头
while cap.isOpened():
	flag, frame = cap.read()
	if not flag:
		break
	else:
		cv2.imshow('video',frame)
		if cv2.waitKey(10)&0XFF == ord('q'):
			break
# 释放资源
cap.release()
cv2.destroyAllWindows()
```

## 视频录制
- [class] cv2.VideoWriter
	- 输出文件对象
	- 输出文件格式
	- 帧率
	- 分辨率
```python
cap = cv2.VideoCapture(0)
fourcc = cv2.Videowrter_fourcc(*'mp4v')
# 分辨率必须符合，否则报错
vw = cv2.Videowriter('output.mp4',fourcc,20,(640,480))

while cap.isOpened():
	ret , frame = cap.read()
	if not ret:
		break
	vw.write(frame)
	cv2.imshow('frame',frame)
	
	if cv2.waitKey(1)&0XFF == ord('q'):
		break

vw.release()
cap.release()
cv2.destroyAllWindows()
```


# 控制鼠标
---
- cv2.setMouseCallback
	- winname
	- callback
	- userdata

# 基础图片处理
----
## ROI
- 切片剪出一定区域
- cv2.split：拆分颜色通道
	- `b,g,r = cv2.split(img)`
- cv2.merge：混合颜色通道
	- `img = cv2.merge((b,g,r))`


## 边界填充
- cv2.copyMakeBorder
	- img：原始图像
	- top_size,bottom_size,left_size, right_size：四周填充宽度
	- borderType：填充类型
		- cv2.BORDER_REPLICATE：复制边缘
		- cv2.BORDER_REFLECT：反射
		- cv2.BORDER_REFLECT_101：反射（不复制边缘像素）
		- cv2.BORDER_WRAP：循环
		- cv2.BORDER_CONSTANT：常量

# 图片运算
---
- 重载了'+'，可以进行标量加法和矩阵加法
	- 数据类型为uint8
	- 运算后自动%256
- cv2.add(mat1, mat2)
	- 越界取255，不取余
- cv2.addWeighted：按权加和
	- mat1,weight1
	- mat2,weight2
	- bias
- cv2.resize：修改大小
	- src：原始图像
	- size：目标大小
	- fx，fy：放缩倍数。若启用，设置size=(0,0)，否则无效

# 图片复制
- self.copy


# 阈值操作
---
- ret, dst = cv2.threshold(src, thresh, maxval, type)
	- src：输入
	- ret:：是否成功
	- dst：输出图
	- thresh：阈值
	- maxval：当像素超过（小于）阈值时赋予的值
	- type：阈值类型
		- cv2.THRESH_BINARY：超过阈值取maxval，否则取0
		- cv2.THRESH_BINARY_INV：binary的反转
		- cv2.THRESH_TRUNC：大于阈值则设为阈值，否则不变
		- cv2.THRESH_TOZERO：大于阈值不变，否则为0
		- cv2.THRESH_TOZERO_INV：大于阈值设为0，否则不变

# 滤波
---
- cv2.blur：均值滤波
	- src：原始图片
	- kernel_size：核大小
- cv2.boxFilter：方框滤波
	- src：原始图片
	- channel：颜色通道；-1表示所有
	- kernel_size：核大小
	- normalize：是否归一化
- cv2.GuassianBlur：高斯滤波
	- 依照高斯分布给予核中各像素权值
	- src
	- ksize
- cv2.medianBlur：中值滤波
	- src
	- ksize

# 形态学运算
---
- cv2.erode：腐蚀
	- src
	- kernel
	- iteration：迭代次数
- cv2.dilate：膨胀
- cv2.mophologyEx：形态学运算
	- src
	- type
		- cv2.MORPH_OPEN：开运算，先膨胀后腐蚀
		- cv2.MORPH_CLOSE：闭运算，先腐蚀后膨胀
		- cv2.MORPH_GRADIENT：梯度运算，膨胀减腐蚀
		- cv2.MORPH_TOPHAT：礼帽，开减原始，剩下刺
		- cv2.MORPH_BLACKHAT：黑帽，原始减闭，剩下沟
	- kernel
- 


# 算子
---
## Sobel算子
$$
G_{x}=\begin{bmatrix}
-1 & 0 & +1 \\
-2 & 0 & +2 \\
-1 & 0 & +1
\end{bmatrix}
\qquad
G_{y}=\begin{bmatrix}
-1 & -2 & -1 \\
0 & 0 & 0 \\
+1 & +2 & +1
\end{bmatrix}
$$
 - 比较两侧像素差异
 - cv2.Sobel
	 - src
	 - ddepth：图像深度；-1自动匹配
		 - cv2.CV_64F：增多位数，能够容纳负数
	 - dx
	 - dy
	 - ksize：Sobel核大小；-1代表默认值3
- 由于算子是右边减左边，所以会出现负数，显示为黑色。
- 利用cv2.convertScaleAbs取绝对值
- 建议分开x、y再求和，不建议同时dx=0，dy=0
```python
sobelx=cv2.Sobel(img,cv.CV_64F,1,0)
sobely=cv2.Sobel(img,cv.CV_64F,0,1)
sobelxy=cv2.addWeighted(sobelx,0.5,sobely,0.5,0)
cv.imshow('sobel',sobelxy)
```
## Scharr算子
$$
G_{x}=\begin{bmatrix}
-3 & 0 & 3 \\
-10 & 0 & 10 \\
-3 & 0 & 3
\end{bmatrix}
\qquad
G_{y}=\begin{bmatrix}
-3 & -10 & -3 \\
0 & 0 & 0 \\
3 & 10 & 3
\end{bmatrix}
$$
- 比Sobel更敏感

## Laplacian算子
- Sobel二阶导
$$
G=\begin{bmatrix}
0 & 1 & 0 \\
1 & -4 & 1 \\
0 & 1 & 0
\end{bmatrix}
$$


# Canny边缘检测
---
1. 高斯滤波
2. 计算梯度和方向
3. 非极大值抑制，消除边缘检测带来的杂散响应
4. 双阈值检测确定真实和潜在的边缘
5. 抑制孤立弱边缘


## 梯度和方向
- 利用Sobel算子滤波
- $G=\sqrt{ G_{x}^2+G_{y}^2 }$
- $\theta = \arctan(G_{y}/G_{x})$

## 非极大值抑制
- 真实边缘法线上的多个像素都可以成为待定边缘像素
- 顺着法线查找梯度极大值，即为最可能的边缘像素
- 法线往往不是垂直或水平的，那么对应的位置可能不是实际存在的像素点，有多种方法来近似该位置梯度。
### 线性插值法
- 通过两侧的像素来线性近似该点梯度
![[OpenCV-非极大值抑制-线性插值法]]
- 考虑周围的3乘3的方格，作中心像素的梯度线，与边界线交于两个亚像素
- 两个亚像素的梯度无法直接求出，我们利用亚像素两侧的像素来估计
- 记锚点为P0，上方为P1，右上方为P2，亚像素为a，G(P0)代表P0梯度
$$
\begin{align}
& \lambda=\frac{d_{1}}{d_{1}+d_{2}}=\cot \theta \\
& G(a) \approx \lambda G(P_{1})+(1-\lambda)G(P_{2}) \\
\end{align}
$$
- 同理求出G(b)
- 若G(P0)不是极大值，则舍弃


### 八向近似法
- 离散为八个方向

## 双阈值检测
- 设定minVal，maxVal
- 梯度值>maxVal：处理为边界
- minVal < 梯度值 < maxVal：连有边界则保留，否则舍弃
- 梯度值 < minVal：舍弃








# 车流量计数项目
---

## 步骤总述
- 视频加载
	- 去除背景
- 车辆识别
	- 基本图像运算
	- 形态学
- 车辆统计
- 加载信息


## 去除背景
- MOG算法


