
- 基于pytorch的边缘检测


# 数学基础
---
## 图像滤波
- 类似于padding的卷积，不改变图像分辨率
- 根据滤波目的可分为
	- 平滑滤波
	- 锐化滤波
- 根据计算方式可分为
	- 线性滤波
	- 非线性滤波
## Roberts算子
$$
d_{x}=\begin{bmatrix}
-1  & 0 \\
0 & 1
\end{bmatrix}
\qquad
d_{y}=\begin{bmatrix}
0 & -1 \\
1 & 0
\end{bmatrix}
$$
- 交叉微分算法
- 适用与45度边缘

## Prewitt算子
$$
d_{x}=\begin{bmatrix}
-1  & 0 & 1 \\
-1  & 0 & 1 \\
-1 & 0 & 1
\end{bmatrix}
\qquad
d_{y}=\begin{bmatrix}
1 & 1 & 1 \\
0 & 0 & 0 \\
-1 & -1 & -1
\end{bmatrix}
$$


## Sobel算子
