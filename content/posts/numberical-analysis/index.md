---
title: "Numberical Analysis Exam"
enableLaTeX: true
tags: [Math, Examination]
date: 2022-05-07T18:04:58+08:00
---

![qingdao](qingdao-tzjt.JPG)

# 考试大纲
:dart: To Reader: 

This blog is JUST FOR EXAMINATION! If you are interested in numberical analysis, please quit this web.
I try to sort out the knowledge points of the course, just to pass the exam.

Based on the course of Professor Zhong Erjie of UESTC.

:anger: **I hate mathematics!**



&nbsp;
## 第二章 非线性方程/方程组的求解
### 1. 二分法及迭代
* 二分法误差估计定理 

### 2. 不动点迭代
* 不动点及不动点迭代的概念
* 迭代格式的选择? 是否收敛?
* 迭代的初值是否合适?

### 3. 牛顿法解非线性方程
> 背景: 如果函数`f(x)`是线性的, 那么它的求根问题就会简化. 牛顿法实质上是一种线性化方法, 将非线性方程逐步归结为某种线性方程来求解.

牛顿法的迭代格式:
$$
    x^{k+1} = x^k - \frac{f(x^k)}{f^'(x^k)}
$$

### 4. 弦截法
> 背景: 弦截法是牛顿法的一个改进. 牛顿法求根时需要计算`f'(x)`, 而导数的计算往往困难. 弦截法使用*差商*来回避导数的计算.

### 5. 收敛阶

### 6. 非线性方程组的牛顿迭代格式
* 雅可比矩阵是什么?



&nbsp;
## 第三章 直接法解线性方程组
### 1. Gauss消元法
求解过程的算法复杂度为`O(n^2)`, 消元过程的算法复杂度为`O(n^3)`.

### 2. 直接三角分解法(Doolittle分解法)
> 背景: `直接`意味着可以由A的元素直接计算L和U, 不需要任何的中间步骤.

一旦L和U得到, 求解`Ax=b`就可以等价表示为求解两个三角形方程组:
1. Ly=b, 求y
2. Ux=y. 求x


&nbsp;
## 第四章 迭代法解线性方程组
> 背景: 对于线性方程组`Ax=b`, 当A为低阶稠密矩阵时, [选主元消去法]是求解的有效方法.  
但是实际情况中A大都是*巨型的稀疏矩阵*, 这是采用迭代法来求解是合适的. 迭代法可以利用A中有大量零元素的特点.

* 迭代法*不一定*最终能够逼近方程组的解, 认识误差向量的概念.

### 1. Jacobi迭代
[雅可比迭代格式和收敛性的判别](https://www.bilibili.com/video/BV14F411e7Ji?p=2&spm_id_from=pageDriver)

[快速计算Bj的特征值](https://www.bilibili.com/video/BV14F411e7Ji?p=4&t=76.0)

[Jacobi迭代由A直接看出Bj](https://www.bilibili.com/video/BV14F411e7Ji?p=4&t=268.1)

### 2. Seidel迭代
[Seidel迭代收敛格式和收敛性的判别](https://www.bilibili.com/video/BV14F411e7Ji?p=6)

Seidel迭代独有的判断收敛性的方式: 若A为`对称阵`, 且A`正定`, 那么迭代收敛.

&nbsp;
## 第五章 插值法
### 1. 插值方法与插值问题
> 背景: 仅已知某些点和该点的函数值的情况下, 如何模拟一个插值函数`P(x)`, 使得误差最小.

* 什么是插值函数P(x)? 被插函数? 插值节点? 插值余项?
### 2. 多项式插值
* 可证明多项式P(x)存在唯一.
* 多项式插值通过解方程组就能得到解`(a0, a1,..., an)`.
### 3. 拉格朗日插值公式
> 背景: 虽然上面的多项式插值能否解决n+1个点的光滑函数, 且解是唯一的. 但是**解方程组是很麻烦的**.

拉格朗日插值公式:
$$
    L_n(x) = l_0(x)y_0 + l_1(x)y_1 + \dots + l_n(x)y_n
$$
插值基函数:

插值条件(插值系数):
$$
    y_0 = f(x_0), y_1 = f(x_1), \dots,y_n = f(x_n)
$$

误差余项Rn(x)

### 4. 牛顿插值公式
>背景: 给定5个插值节点及其函数值, 可以得到L4(x); 由于某种原因, 需要加入一个新的插值节点. Lagrange插值法之前的计算结果(l)均失效, 需要重新计算. 非常的不方便.
* 牛顿法是基于**差商**的概念. 导数是差商的极限.
* 差商的差商是高阶差商.

牛顿插值法的插值函数(以二次插值举例):
$$
    P(x) = a_0 + a_1(x-x_0) + a_2(x-x_0)(x-x_1)
$$
需要做的就是解出系数`a0,a1,...`.

所以引入差商的符号:
$$
    a_1=f[x_0,x_1]=\frac{f(x_1)-f(x_0)}{x_1-x_0}
$$
$$
    a_2=f[x_0,x_1,x_2]=\frac{f[x_1,x_2]-f[x_0,x_1]}{x_2-x_0}
$$


### 5. Hermite插值
> 背景: 有时我们已知的条件不都是函数值, 也有导数值. 例如已知两个点的函数值和两个点的导数值, 可以应用Heimite插值法得到三次多项式.

求Hermite插值函数的方法: 构造差商表, **重复节点特殊处理**.

Hermite插值方法的余项证明与Langrange插值法相同.

### 6. 分段低次插值
>背景: 次数太高的多项式插值的效果不好. 比如龙格现象.

* 分段: 把被插值函数所在的大区间分成一个个的小区间.
* 低次: 每个小区间上用次数不超过`3`的函数来逼近
#### 6.1 分段线性插值
就是分段折线

分段线性插值的优点:
1. 简单
2. 当二阶导数趋近0时, 一定收敛

分段线性插值的缺点:
1. 分段折线不光滑, 分段点处不能求导.

#### 6.2 分段Hermite插值
>背景: 为了解决分段线性插值的缺点(存在尖点).

已知函数在(n+1)个点的函数值值以及其导数值, 去构造一阶连续可导函数.

分段Hermite插值根据(n+1)个已知点划分为(n+1)个区间. 这样在每个小区间上都已知`4`个条件, 可以使用`3`次Hermite插值.

结论: 已知(2n+2)个条件的情况下, 居然只得到**一阶连续可微函数**. 结论太差!

&nbsp;
## 第六章 拟合
:mag: **插值, 拟合, 逼近的区别**
#### 1. 最佳平方逼近

#### 2. 最小二乘法
>背景: 已知不共线的三点, 如何确定一条**可信**的直线.

三个点可以用插值来模拟二次多项式, 但题目要求了用一次多项式, 这是插值无法做到的.

不共线的三点不可能同时经过一条直线, 所以要用逼近的思想. 找一条近似的直线, 使得**误差**最小.
* **与插值的区别**: 插值是明确给出`n+1`个插值条件, 得到`n`次多项式.
* **如何定义误差最小?**: 函数间的距离.

### 1. 线性拟合
拟合的函数是`n`次多项式, 可转化为超定方程`GX`.
* 其中规定`G`为系数矩阵, `X`为变量的列向量.
* 同时定义列向量`F`为给出的函数值.
* `GX=F`是超定方程组, 没有准确解. 得到残差最小的解的方法即**最小二乘法**.

所以线性拟合的残差`r = |GX - F|`, 而找到目标函数的宗旨就是使`r`最小. 使用**初等变分原理**将这个问题转化为**正规方程组**求解的问题.


&nbsp;
## 第七章 数值积分
>背景: 定积分的计算中可能无法找到原函数的情况. 考虑定积分的本质是一句具体的数, 我们的目标就是找到这个数的近似值, 越接近越好.

解决的两种思路: `积分中值定理` 和 `插值型求积公式(近似被积函数)`.

### 1. 积分中值定理
基本的积分中值定理:
$$
    \int_{a}^{b}f(x)dx = f(\xi)(b-a)
$$

将一个区域的面积转化为矩形的面积. 如何确定矩形的高呢? `左矩阵`, `右矩阵`, `中间矩阵`, `梯形公式`.

更常用的积分公式是 在乘积函数积分中, 如果`g(x)`不变号, 则有: 
$$
    \int_{a}^{b}g(x)f(x)dx =f(\xi)\int_{a}^{b}g(x)dx 
$$

### 2. 插值型求积公式
在被积函数很复杂的情况下, 可以对其进行近似处理, 例如使用`Lagrange插值法`.
#### 二次插值: Simpson公式
取二次插值的步长`h=(b-a)/2`, 即增加一个插值节点`(b-a)/2`, Simpson公式化简的结果为:
$$ 
    \int_{a}^{b}f(x)dx = \frac{b-a}{6}[f(a)+4f(\frac{a+b}{2})+f(b)]+R[f]
$$


:pushpin: Simpson公式满足`3`阶代数精度. 虽然它只是二次插值得到的.


### 3. 余项
* 插值型求积公式的余项, 即对应的插值方法(如Lagrange, Newton)的余项在区间上的积分.
* 梯形公式方法的余项可以用**积分中值定理**来优化.
* Simpson公式的余项**不能**使用积分中值定理来优化, 因为不满足保号的条件.

### 4. 衡量求积公式的好坏
**代数精度**: 不是一种误差, 而是对误差的描述.

如何得知某个公式的代数精度: 只要带入一个m次多项式验证余项是否为0即可.

### 5.复合求积公式
为了提高精度通常把积分区间分为若干个子区间, 再在每个子区间上应用低阶求积公式.

* `复合梯形公式`: 将区间等分.
* `复合simpson公式`: 将区间偶数等分.

&nbsp;
## 第八章 常微分方程初值问题数值解法
将研究的内容进一步限定为: `一阶初值问题`, `单步法`.
>背景: 在无法给出解析表达式时如果利用数值方法求出`y`的近似解?


### 1. 简单的数值方法
#### 1.1 Euler公式
使用`一阶向前差商`近似替代`y'`. 得到递推的数列表达式:
$$
    y_{n+1} = y_{n} + hf(x_n,y_n), n=0,1,2,...
$$

**误差**: Euler法使用的近似代替只有`一阶精度`, 所以误差很大. 此时有两种解决方案:
1. 加细步长`h`, 若不行再加细. 总是能得到正确的, 如果你不嫌弃带来的计算变得缓慢的问题.
2. 换方法.

#### 1.2 梯形公式

> 背景:为得到比Euler法精度更高的计算公式. 梯形公式具有`二阶`精度.

对`y' = f(x,y)`的两端进行**局部的**积分, 然后用梯形公式近似计算右边.

#### 1.3 改进Euler公式

先用欧拉公式求得一个`近似的yn+1`, 带入梯形公式, 得到`矫正的yn+1`.

