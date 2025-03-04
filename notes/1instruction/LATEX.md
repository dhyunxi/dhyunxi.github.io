
# MATRIX
- 使用\begin{}和\end{}来标识起止
	- matrix是无括号
	- bmatrix方括号
	- Bmatrix花括号
	- pmatrix圆括号
	- vmatrix行列式
	- Vmatrix双竖线






$$
\begin{bmatrix}
asd\\asd&asd
\end{bmatrix}

\begin{Bmatrix}
asd
\end{Bmatrix}


$$
$$

\left(
\begin{array}
asd
\end{array}
\right)

$$




# FUNCTION
- 分段函数cases
- \{aligned} 排版环境，允许换行符和分段符
- \quad-----表示的是一个字体的宽度
\qquad-------表示的是产生2个当前字体的缩进。
\enspace-----表示的是产生0.5当前字体的缩进
- equation里只能用aligned不能align
$$
\begin{equation}
y(x)=\left\{
    \begin{aligned}
	-1 \quad x<0\\
	0 \quad x=0\\
	1 \quad x>0\\
	\end{aligned}
	\right.
	
\end{equation}

$$


$$
\begin{equation}
y(x)=\left\{
\begin{aligned}

-1x<0x \quad +1x=0 

\end{aligned}
\right.
\end{equation}
$$



# 等号
- 使用\stackrel{上位符号}{基位符号}来在等号上添加文字
$$
\stackrel{\mathrm{def}}{=}
$$
- \triangleq 三角等号
$$\triangleq$$



- [Latex数学符号速查（Latex math symbols） - 知乎](https://zhuanlan.zhihu.com/p/686538247)
- xx 可以打出叉乘
- approx约等于$\approx$



# 特殊字体
---
- 罗马字体$\mathrm{RmFont}$
	- rm 
	- \mathrm
- 斜体$\it{}asd$
- 黑板粗体  $\mathbb{R}$
	- RR、NN
	- \mathbb
- 花体   $\mathcal{L}$
	- LL、HH
	- \mathcal
	- 只适用于大写字母
- 斜体  $\mathit{123asd}$
	- \it
	- \mathit
- 印刷体$\mathtt{123asd}$
	- \mathtt
- 印刷体$\mathsf{asd123}$
	- \mathsf
- 函数名$\operatorname{operator F}$
	- \operatorname

# 特殊符号
---
- 花体小l   $\ell$
	- ell
	- \ell
- 花体R    $\Re$
- 微分算子  $\nabla$
	- nabl
	- \nabla
- 连加  $\sum$
	- sum  **区分于Sigma
	- \sum
- 连乘 $\prod$
	- prod
	- \prod
- 无穷 $\infty$
	- ooo
	- \infty
- 空格 ~
	- \space