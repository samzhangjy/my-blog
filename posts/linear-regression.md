---
title: 浅谈线性回归
desc: 线性回归即 y = wx + b ，是深度学习中的最基本组成元件。
cover: /assets/posts/linear-regression.jpg
date: 2025.07.20
tags:
  - 深度学习
---

## 线性回归

**回归（regression）** 即通过建模某个或多个自变量与因变量之间关系，预测某一数值的问题。回归问题预测的数值是连续的；分类则是预测离散数值。

线性回归则是假设自变量 $\mathbf{x}$ 与因变量 $y$ 之间具有线性关系，且观测数据噪声遵循正态分布。一般地，我们把试图预测的目标成为**标签（label）**或**目标（target）**，预测所依据的自变量（数据）称为**特征（feature）**或**协变量（covariate）**。一般使用 $n$ 表示数据集样本容量，对于索引为 $i$ 的样本，其输入表示为 $\mathbf{x}^{(i)} = [x_1^{(i)},x_2^{(i)}, \cdots]^{\top}$ ，对应标签为 $y^{(i)}$ 。

### 线性模型

首先，给出线性模型的一般定义式：
$$
\hat{y} = \mathbf{w}^{\top}\mathbf{x} + b.
$$
其中 $\mathbf{w}$ 表示模型**权重（weight）**，$b$ 表示**偏置（bias）**，$\mathbf{x}$ 表示单个数据样本的特征。对于特征矩阵 $\mathbf{X} \in \R^{n \times d}$ ，我们同样有：
$$
\hat{\mathbf{y}} = \mathbf{Xw} + b.
$$
线性模型中，偏置项的意义在于提升模型的表达能力，其物理意义即为当所有特征取值都为 0 时，预测值应该为多少。上述二式即代表对于输入特征的**仿射变换（affine transformation）**，即通过加权对特征进行**线性变换（linear transformation）**，并通过偏置项进行**平移（translation）**。

> 矩阵乘法即对于张空间内的向量进行变换。

我们的任务就是对于给定的特征数据集，找到一组最优的 $\mathbf{w}, b$ 来拟合给定的标签，即寻找最优模型参数。

### 损失函数

**损失函数（loss function）**能够量化目标的实际值与预测值之间的差距。通常我们有损失 $\geq 0$ ，且数值越小损失越小，完美预测时损失为 0 。回归问题中最常用的损失函数为**平方损失函数（Square Error Function）**：
$$
l^{(i)}(\mathbf{w}, b) = \frac{1}{2}(\hat{y}^{(i)} - {y}^{(i)})^2.
$$
常数 $\frac{1}{2}$ 的意义在于使 $l$ 的导数常数系数为 1 。对于整个数据集，我们有：
$$
L(\mathbf{w}, b) = \frac{1}{n} \sum^n_{i = 1}l^{(i)}(\mathbf{w}, b) = \frac{1}{n}\sum^{n}_{i = 1}\frac{1}{2}(\mathbf{w}^{\top}\mathbf{x}^{(i)} + b - y^{(i)})^2 .
$$

### 随机梯度下降

尽管对于线性回归，我们可以通过最小化 $\Vert\mathbf{y} - \mathbf{Xw}\Vert^2$ 求出其解析解 $\mathbf{w}^* = (\mathbf{X}^{\top}\mathbf{X})^{-1}\mathbf{X}^{\top}\mathbf{y}$ ，但在大部分深度学习模型中，我们无法简单通过解析解来获得最优参数。**梯度下降（gradient descent）**正是一中可以通过不断地在损失函数递减的方向上更新参数来降低误差的方法。

梯度下降的一般定义即是在迭代时计算损失函数均值关于模型参数的偏导数（通过链式法则，也称梯度），但效率极低。为了避免在每一次更新时遍历一遍数据集，我们通常会在每次需要更新的时候随机抽取一小批样本，即**小批量随机梯度下降（minibatch stochastic gradient descent）**。

在每次迭代时，我们首先随机抽样一个容量固定的小批量 $\mathcal{B}$ ，并计算小批量的平均损失关于模型参数的偏导数。最后，我们将梯度乘以一个预先确定的正数 $\eta$（**学习率，learning rate**），并从当前参数的值中减掉。

形式化地，对于每次迭代，我们有：
$$
(\mathbf{w}, b) \larr (\mathbf{w}, b) - \frac{\eta}{\vert\mathcal{B}\vert} \sum_{i \in \mathcal{B}}\partial_{(\mathbf{w}, b)}l^{(i)}(\mathbf{w}, b) .
$$
对于类似 $\eta$ 和 $\vert\mathcal{B}\vert$ （**批次大小，batch size**） 的可以调整但是不在训练过程中的参数我们成为**超参数（hyperparameter）**。**调参（hyperparameter tuning）**即选择超参数的过程。

需要注意的是，即便函数完全遵循线性回归且无噪声，模型参数 $\hat{\mathbf{w}}$ ，$\hat{b}$ 也不会达到实际最小值，而是无限接近最小值。对于较为复杂的深度神经网络，损失平面通常包含多个最小值。实践上一般不会追求训练集上的最小损失，而是追求能在陌生数据上实现较小损失。这一过程即**泛化（generalization）**。

### 噪声与正态分布

若随机变量 $x$ 具有均值 $\mu$ 和方差 $\sigma^2$ ，其正态分布概率密度函数如下：
$$
p(x) = \frac{1}{\sqrt{2\pi\sigma^2}}\exp(-\frac{1}{2\sigma^2}(x - \mu)^2) .
$$
改变均值会产生沿 $x$ 轴的偏移，增加方差会分散分布、降低峰值。

而均方误差损失函数可以用于线性回归的原因之一是我们假设了观测中包含噪声，而噪声服从正态分布：
$$
y = \mathbf{w}^{\top}\mathbf{x} + b + \epsilon.
$$
通过给定的 $\mathbf{x}$ 观测到特定 $y$ 的**似然（likelihood）**为：
$$
p(y \mid \mathbf{x}) = \frac{1}{\sqrt{2\pi\sigma^2}}\exp(-\frac{1}{2\sigma^2}(y - \mathbf{w}^{\top}\mathbf{x} - b)^2) .
$$
对于整个数据集的似然，即为：
$$
P(\mathbf{y} \mid \mathbf{X}) = \prod^n_{i = 1}p(y^{(i)}\mid \mathbf{x}^{(i)}) .
$$
通过最小化负对数似然，得：
$$
-\log P(\mathbf{y} \mid \mathbf{X}) = \sum^n_{i = 1}\frac{1}{2}\log(2\pi\sigma^2) + \frac{1}{2\sigma^2}(y^{(i)} - \mathbf{w}^{\top}\mathbf{x}^{(i)} - b)^2 .
$$
剔除式中常数项，该式即等价于最小化均方误差。