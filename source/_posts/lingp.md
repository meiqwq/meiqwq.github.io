---
title: 线性规划对偶
date: 2024-03-11 19:49:36
tags: [计算理论,线性规划]
mathjax: true
---
# 对偶问题
考虑一个标准的线性规划问题：

$max:z=\sum\limits_{i=1}^{n}c_i\cdot x_i$

限制如下:
$$\begin{cases} \sum\limits_{i=1}^{n}a_{1,i}\cdot x_i \leq b_1 \\ \sum\limits_{i=1}^{n}a_{2,i}\cdot x_i \leq b_2 \\ .....\\\sum\limits_{i=1}^{n}a_{m,i}\cdot x_i \leq b_m\end{cases}$$

$\forall i \in [n] ,x_i \geq0$

它的对偶形式是：

$min:z'=\sum\limits_{j=1}^m b_j\cdot y_j$

限制：

$$\begin{cases} \sum\limits_{j=1}^{m}a_{j,1}\cdot y_j \geq c_1 \\ \sum\limits_{j=1}^{m}a_{j,2}\cdot y_j \geq c_2 \\ .....\\ \sum\limits_{j=1}^{m}a_{j,n}\cdot y_j \geq c_n\end{cases}$$

$\forall j \in [m],y_j \geq 0$

用矩阵的语言就是：

prime LP:

$max:z=\vec{c}^T\cdot \vec{x}$

$A\cdot \vec{x} \preceq \vec{b}$

$\vec{x} \succeq 0$

dual LP:
$min:z'=\vec{b}^T\cdot \vec{y}$


$A^T\vec{y} \succeq \vec{c}$

$\vec{y} \succeq$

# weak duality
上述两个LP,的任意一组feasible solution $\hat{x},\hat{y}$,有$\vec{c}^T\cdot \hat{x} \leq \vec{b}^T \cdot \vec{y}$,i.e.最大化一定小于等于最小化.

证:

由于: $A^T \cdot \hat{y} \succeq \vec{c}$并且$\hat{x} \succeq 0$,因而:$\vec{c}^T\cdot \vec{x} \leq (A^T\cdot \hat{y})^T \hat{x}=\hat{y}^T\cdot A \cdot \hat{x}$

同样地：因为 $A\cdot \hat{x} \preceq \vec{b}$,$\hat{y} \succeq 0$,因而 $\vec{b}^T\cdot \hat{y} \geq (A \cdot \hat{x})^T \cdot (\hat{y}^T)^T=(\hat{y}^T\cdot A \cdot \hat{x})^T$,注意到 $\hat{y}^T\cdot A \cdot \hat{x}$是一个实数，所以 $(\hat{y}^T\cdot A \cdot \hat{x})^T=\hat{y}^T\cdot A \cdot \hat{x}$

因而：

${\vec{c}}^{T} \cdot \hat{x} \leq {\hat{y}}^{T} \cdot A \cdot \hat{x} \leq {\vec{b}}^{T} \cdot \hat{y} $

# 互补松弛
假设 $\hat{x},\hat{y}$是两个LP的最优解，有$$\vec{c}^T \cdot \hat{x} = \hat{y}^T\cdot A \cdot \hat{x} = \vec{b}^T \cdot \hat{y}$$


我们对其变形：
$$\hat{y}^T\cdot A \cdot \hat{x}-\vec{c}^T \cdot \hat{x} = 0$$
$$(\hat{y}^T\cdot A-\vec{c}^T)\cdot \vec{x} = 0$$

由于 $\hat{y}^T\cdot A-\vec{c}^T=(A^T\cdot \hat{y}-\vec{c})^T$,i.e. $(A^T\cdot \hat{y}-\vec{c})^T \cdot \hat{x} =0$

写成代数形式：$\sum\limits_{i=1}^n[({\sum\limits_{j=1}^ma_{j,i}\cdot \hat{y}_j})-c_i]\cdot \hat{x}_i = 0$

这说明了：如果$x_i$非0，那么对偶问题中它对应的不等式一定取```=```，如果一个不等式不取```=```,那么对偶问题中这个不等式对应的变量一定为```0```

