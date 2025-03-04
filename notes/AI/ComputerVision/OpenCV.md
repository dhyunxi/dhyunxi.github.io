
#CV 

# 目录
---
- Read Images Videos and Webcams
- Basic Functions
- Resizing and Croppiung
- Shapes and Text
- Warp Prespective
- Joining Images
- Color Detection
- Contour/Shape Detection
- 
- Face Detection


# What is Images
---
- 图像是像素的集合
- 像素是通道的组合
- 2 Level ：只有黑白两色
- 灰度图：8bit，256Levels，254种灰度
- 彩色图：3个灰度图分别代表RGB通道

- cv2.VideoCapture( video_dir:str )
	- 返回一个视频对象，需要用read循环读取
	- read返回两个参数，分别是读取是否成功和帧
```python
cap = cv2. VideoCapture( ... )
while True:
	success, img = cap.read()
	cv2.imshow('Video' , img)
```

# 方法与函数
---
## 窗口
- cv2.namedWindow：创建命名窗口
	- winname：窗口名称
	- flags：
		- 
- cv2.imshow：显示窗口
- cv2.destroyAllwindows：摧毁所有窗口
- cv2.resizeWindow：改变窗口大小
- cv2.waitKey：等待

## 图像
### IO
- cv2.imread：读入一个图片文件
	- params：
		- file：文件路径
		- flags：cv::ImreadModes, 用于设定读取模式
			- cv2.IMREAD_GRAYSCALE：转换为灰度图
	- 若打开失败，返回NULL矩阵
	- 彩色图片通道以B，G，R顺序储存
- cv2.imshow
	- params：
		- winname：窗口名称
		- mat：要展示的图片
	- 会被压缩到8位256色
	- 若当前没有窗口，会创建适应图片大小的窗口
	- 需搭配waitKey使用
- cv2.waitKey
	- params
		- delay：延迟的时间（毫秒）
			- 0：保持窗口直到任何按键按下
			- 正整数：保持以毫秒为单位的时间
	- 在延迟期间，若键盘有输入，则返回输入ASCII，否则返回-1

### 修改
- cv2.cvtColor：色彩转换
- cv2.GaussianBlur：高斯模糊
- cv2.Canny：边缘检测
- cv2.dilate：膨胀边缘
- cv2.erode：缩减边缘
- cv2.resize：缩放
- cv2.crop

### 实例参数
- self.shape



## 视频
### VideoCapture
- VideoCapture是cv2中定义的类，用来载入视频或摄像头
#### 实例参数
- file or ID：文件路径或摄像头ID

#### 实例方法
- set


